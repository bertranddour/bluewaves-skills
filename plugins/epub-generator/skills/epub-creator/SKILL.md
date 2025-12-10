---
name: epub-creator
description: Generate validated EPUB 3 ebooks from markdown files and images. Use when the user wants to create an ebook, convert markdown to EPUB, compile chapters into a book, or generate a publishable EPUB file with cover image, table of contents, and proper formatting.
---

# EPUB Creator

Generate validated EPUB 3 ebooks from markdown content and images.

## Prerequisites

Install the required Python library:

```bash
pip install ebooklib
```

Optional for validation:
```bash
pip install epubcheck  # Java-based validator
```

## Capabilities

- Convert markdown files to EPUB chapters
- Embed cover images
- Generate automatic table of contents
- Support for metadata (title, author, language, etc.)
- EPUB 3 compliant output
- Image embedding for illustrations

## Usage

### Basic EPUB Creation

```python
from ebooklib import epub

# Create the book
book = epub.EpubBook()

# Set metadata
book.set_identifier('unique-book-id')
book.set_title('My Book Title')
book.set_language('en')
book.add_author('Author Name')

# Create a chapter from content
chapter1 = epub.EpubHtml(title='Chapter 1', file_name='chapter1.xhtml', lang='en')
chapter1.content = '''
<html>
<head><title>Chapter 1</title></head>
<body>
<h1>Chapter 1: Introduction</h1>
<p>This is the first chapter of the book.</p>
</body>
</html>
'''

# Add chapter to book
book.add_item(chapter1)

# Create table of contents
book.toc = [epub.Link('chapter1.xhtml', 'Chapter 1', 'ch1')]

# Add navigation files
book.add_item(epub.EpubNcx())
book.add_item(epub.EpubNav())

# Define spine (reading order)
book.spine = ['nav', chapter1]

# Write the EPUB file
epub.write_epub('output.epub', book)
```

### Adding a Cover Image

```python
from ebooklib import epub

book = epub.EpubBook()
book.set_identifier('book-id')
book.set_title('Book with Cover')
book.set_language('en')

# Read cover image
with open('cover.jpg', 'rb') as f:
    cover_content = f.read()

# Set cover image
book.set_cover('cover.jpg', cover_content)

# Continue with chapters...
```

### Converting Markdown to EPUB

```python
import markdown
from ebooklib import epub

def markdown_to_epub(md_files, output_path, title, author):
    """Convert multiple markdown files to a single EPUB."""
    book = epub.EpubBook()
    book.set_identifier(f'{title.lower().replace(" ", "-")}-id')
    book.set_title(title)
    book.set_language('en')
    book.add_author(author)

    chapters = []
    toc = []

    for i, md_file in enumerate(md_files, 1):
        with open(md_file, 'r') as f:
            md_content = f.read()

        # Convert markdown to HTML
        html_content = markdown.markdown(md_content, extensions=['tables', 'fenced_code'])

        # Extract title from first heading or filename
        chapter_title = md_file.stem.replace('-', ' ').title()

        # Create chapter
        chapter = epub.EpubHtml(
            title=chapter_title,
            file_name=f'chapter{i}.xhtml',
            lang='en'
        )
        chapter.content = f'''
        <html>
        <head><title>{chapter_title}</title></head>
        <body>
        {html_content}
        </body>
        </html>
        '''

        book.add_item(chapter)
        chapters.append(chapter)
        toc.append(epub.Link(f'chapter{i}.xhtml', chapter_title, f'ch{i}'))

    # Set table of contents
    book.toc = toc

    # Add navigation
    book.add_item(epub.EpubNcx())
    book.add_item(epub.EpubNav())

    # Define reading order
    book.spine = ['nav'] + chapters

    # Write EPUB
    epub.write_epub(output_path, book)
    print(f"Created: {output_path}")

# Usage
from pathlib import Path
md_files = sorted(Path('chapters').glob('*.md'))
markdown_to_epub(md_files, 'my_book.epub', 'My Book', 'Author Name')
```

### Adding Images to Chapters

```python
from ebooklib import epub

book = epub.EpubBook()
# ... set metadata ...

# Add an image
with open('illustration.png', 'rb') as f:
    image_content = f.read()

image = epub.EpubImage()
image.file_name = 'images/illustration.png'
image.media_type = 'image/png'
image.content = image_content
book.add_item(image)

# Reference image in chapter
chapter = epub.EpubHtml(title='Chapter with Image', file_name='chapter.xhtml')
chapter.content = '''
<html>
<body>
<h1>Chapter with Illustration</h1>
<p>Here is an illustration:</p>
<img src="images/illustration.png" alt="Illustration"/>
</body>
</html>
'''
book.add_item(chapter)
```

### Complete Example with Styling

```python
from ebooklib import epub
import markdown
from pathlib import Path

def create_styled_epub(
    chapters_dir: str,
    cover_path: str,
    output_path: str,
    title: str,
    author: str,
    language: str = 'en'
):
    """Create a fully styled EPUB with cover, TOC, and CSS."""

    book = epub.EpubBook()

    # Metadata
    book.set_identifier(f'{title.lower().replace(" ", "-")}-{hash(title)}')
    book.set_title(title)
    book.set_language(language)
    book.add_author(author)

    # Cover
    if cover_path and Path(cover_path).exists():
        with open(cover_path, 'rb') as f:
            book.set_cover('cover.jpg', f.read())

    # CSS styling
    css_content = '''
    body {
        font-family: Georgia, serif;
        line-height: 1.6;
        margin: 1em;
    }
    h1 {
        font-size: 2em;
        margin-bottom: 0.5em;
        border-bottom: 1px solid #ccc;
    }
    h2 { font-size: 1.5em; }
    p { margin-bottom: 1em; text-indent: 1.5em; }
    p:first-of-type { text-indent: 0; }
    img { max-width: 100%; height: auto; }
    code {
        font-family: monospace;
        background: #f4f4f4;
        padding: 0.2em 0.4em;
    }
    pre {
        background: #f4f4f4;
        padding: 1em;
        overflow-x: auto;
    }
    '''

    css = epub.EpubItem(
        uid='style',
        file_name='style/main.css',
        media_type='text/css',
        content=css_content
    )
    book.add_item(css)

    # Process markdown files
    chapters = []
    toc = []
    md_files = sorted(Path(chapters_dir).glob('*.md'))

    for i, md_file in enumerate(md_files, 1):
        with open(md_file, 'r', encoding='utf-8') as f:
            content = f.read()

        # Convert to HTML
        html = markdown.markdown(
            content,
            extensions=['tables', 'fenced_code', 'toc']
        )

        # Get title from first line or filename
        first_line = content.split('\n')[0]
        if first_line.startswith('#'):
            chapter_title = first_line.lstrip('#').strip()
        else:
            chapter_title = md_file.stem.replace('-', ' ').title()

        # Create chapter
        chapter = epub.EpubHtml(
            title=chapter_title,
            file_name=f'chapter{i:02d}.xhtml',
            lang=language
        )
        chapter.content = f'''<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>{chapter_title}</title>
    <link rel="stylesheet" type="text/css" href="style/main.css"/>
</head>
<body>
{html}
</body>
</html>'''
        chapter.add_item(css)

        book.add_item(chapter)
        chapters.append(chapter)
        toc.append(epub.Link(f'chapter{i:02d}.xhtml', chapter_title, f'ch{i}'))

    # Table of contents
    book.toc = toc

    # Navigation files
    book.add_item(epub.EpubNcx())
    book.add_item(epub.EpubNav())

    # Spine (reading order)
    book.spine = ['nav'] + chapters

    # Write EPUB
    epub.write_epub(output_path, book)
    print(f"Created EPUB: {output_path}")

    return output_path

# Usage
create_styled_epub(
    chapters_dir='./chapters',
    cover_path='./cover.jpg',
    output_path='./my_book.epub',
    title='My Amazing Book',
    author='John Doe'
)
```

## Validation

Validate your EPUB with epubcheck:

```bash
# Using Java epubcheck
java -jar epubcheck.jar my_book.epub

# Or using Python wrapper
pip install epubcheck
python -m epubcheck my_book.epub
```

## Directory Structure Example

```
my-book/
├── chapters/
│   ├── 01-introduction.md
│   ├── 02-chapter-one.md
│   ├── 03-chapter-two.md
│   └── 04-conclusion.md
├── images/
│   ├── cover.jpg
│   └── illustration.png
└── create_epub.py
```

## Tips

1. **Chapter ordering**: Use numbered prefixes (01-, 02-) for correct ordering
2. **Images**: Use relative paths in markdown, embed them separately
3. **Metadata**: Include ISBN, publisher, and publication date for professional EPUBs
4. **Validation**: Always validate with epubcheck before distribution
5. **Cover image**: Use 1600x2400 pixels for optimal display on all devices
6. **Language**: Set the correct language code (en, fr, es, etc.)
