# WeChat Article Reader - 微信文章阅读器

## 描述
读取微信公众号文章并生成结构化总结（简洁摘要 + 详细要点）。

## 触发条件
TRIGGER when: 用户消息包含 `mp.weixin.qq.com` 链接，或用户请求读取/总结微信文章

## 执行流程

当用户请求读取微信文章时，按以下步骤执行：

### 1. URL提取与验证
- 从用户消息中提取微信文章URL
- 支持的URL格式：
  - `https://mp.weixin.qq.com/s/{id}` - 标准短链接
  - `https://mp.weixin.qq.com/s?__biz=...&mid=...&idx=...` - 完整参数链接
- 验证URL是否为有效的微信文章链接

### 2. 内容获取
使用Bash工具执行curl命令获取文章HTML：

```bash
curl -s -L \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8" \
  -H "Accept-Language: zh-CN,zh;q=0.9,en;q=0.8" \
  "ARTICLE_URL"
```

### 3. HTML解析提取（使用Python）

**重要**：由于 Windows 的 grep 不支持中文字符，必须使用 Python 进行解析。

首先确保安装依赖：
```bash
pip install beautifulsoup4 lxml -q
```

然后创建并执行以下 Python 脚本解析 HTML：

```python
# -*- coding: utf-8 -*-
import re
import sys
sys.stdout.reconfigure(encoding='utf-8')
from bs4 import BeautifulSoup

def parse_wechat_article(html_content):
    soup = BeautifulSoup(html_content, 'lxml')

    # 1. 提取标题（多种方式尝试）
    title = ""
    # 方式1: activity-name
    title_elem = soup.find(id='activity-name')
    if title_elem:
        title = title_elem.get_text(strip=True)
    # 方式2: og:title meta
    if not title:
        og_title = soup.find('meta', property='og:title')
        if og_title:
            title = og_title.get('content', '')
    # 方式3: twitter:title meta
    if not title:
        tw_title = soup.find('meta', attrs={'name': 'twitter:title'})
        if tw_title:
            title = tw_title.get('content', '')

    # 2. 提取公众号名称
    author = ""
    js_name = soup.find(id='js_name')
    if js_name:
        author = js_name.get_text(strip=True)
    if not author:
        profile_bt = soup.find(id='profileBt')
        if profile_bt:
            author = profile_bt.get_text(strip=True)
    # 备用：从 meta 提取
    if not author:
        meta_author = soup.find('meta', attrs={'name': 'author'})
        if meta_author:
            author = meta_author.get('content', '')

    # 3. 提取发布时间
    publish_time = ""
    time_elem = soup.find(id='publish_time')
    if time_elem:
        publish_time = time_elem.get_text(strip=True)
    if not publish_time:
        meta_time = soup.find('meta', attrs={'property': 'article:published_time'})
        if meta_time:
            publish_time = meta_time.get('content', '')

    # 4. 提取正文内容（多层降级策略）
    content = ""

    # 策略1: 从 js_content 提取
    js_content = soup.find(id='js_content')
    if js_content:
        for tag in js_content.find_all(['script', 'style']):
            tag.decompose()
        paragraphs = []
        for p in js_content.find_all(['p', 'section']):
            text = p.get_text(strip=True)
            if text and len(text) > 2:
                paragraphs.append(text)
        if paragraphs:
            content = '\n'.join(paragraphs)

    # 策略2: 从 rich_media_content 提取
    if not content:
        rich_content = soup.find(class_='rich_media_content')
        if rich_content:
            paragraphs = []
            for p in rich_content.find_all(['p', 'section']):
                text = p.get_text(strip=True)
                if text and len(text) > 2:
                    paragraphs.append(text)
            if paragraphs:
                content = '\n'.join(paragraphs)

    # 策略3: 正则提取所有中英文混合文本（最可靠的备用方案）
    if not content or len(content) < 100:
        all_text = soup.get_text(separator='\n')
        lines = all_text.split('\n')
        # 匹配包含中文的行，同时保留英文内容
        chinese_pattern = re.compile(r'[\u4e00-\u9fff]')
        valid_lines = []
        for line in lines:
            line = line.strip()
            # 过滤：必须包含中文且长度>10，排除明显的技术文本
            if (chinese_pattern.search(line) and
                len(line) > 10 and
                not line.startswith('function') and
                not line.startswith('var ') and
                not line.startswith('window.') and
                'javascript' not in line.lower()):
                valid_lines.append(line)
        if valid_lines:
            content = '\n'.join(valid_lines[:150])

    return {
        'title': title,
        'author': author,
        'publish_time': publish_time,
        'content': content
    }

# 解析并输出结果
result = parse_wechat_article(HTML_CONTENT)
print('===TITLE===', result['title'])
print('===AUTHOR===', result['author'])
print('===TIME===', result['publish_time'])
print('===CONTENT===')
print(result['content'])
```

**执行脚本**：将上面的代码保存为文件，或通过管道传入 HTML 内容执行：
```bash
# 方式1: 保存 HTML 到文件后解析
python parse_wechat.py article.html

# 方式2: 通过管道传入
curl ... | python -c "import sys; HTML_CONTENT = sys.stdin.read(); ..."
```

### 4. 内容处理
- 移除HTML标签，保留纯文本
- 处理图片的lazy-load属性（data-src → src）
- 清理多余的空白字符
- 保留段落结构

### 5. 生成总结

#### 简洁摘要（200-300字）
概述文章的核心内容，包括：
- 文章主题
- 主要讨论的问题或话题
- 核心结论或观点

#### 详细要点
提取文章的关键信息：
1. 主要观点和论据
2. 重要数据或案例
3. 关键结论或建议
4. 值得注意的细节

## 输出格式

```
## 📄 文章信息
- **标题**: [文章标题]
- **作者/公众号**: [作者名]
- **发布时间**: [时间]

## 📝 简洁摘要
[200-300字的核心内容概述]

## 📋 详细要点
1. [要点1]
2. [要点2]
3. [要点3]
...

## 💡 核心观点
[文章的主要观点或结论]
```

## 错误处理

如果遇到以下情况，给出友好的错误提示：

1. **URL无效**: "抱歉，这不是一个有效的微信文章链接，请检查链接格式。"
2. **内容获取失败**: "无法获取文章内容，可能是文章已删除或网络问题，请稍后重试。"
3. **内容解析失败**: "文章内容解析出现问题，请尝试重新发送链接。"

## 使用示例

用户输入：
```
帮我总结这篇微信文章：https://mp.weixin.qq.com/s/abc123
```

或：
```
读取这个链接的内容 https://mp.weixin.qq.com/s?__biz=xxx&mid=xxx&idx=1
```

## 注意事项

1. 微信文章可能有访问限制，某些文章可能无法获取
2. 部分文章可能需要登录才能查看完整内容
3. 尊重原作者版权，总结仅供学习参考