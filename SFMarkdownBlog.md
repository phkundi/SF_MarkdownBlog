## Shipfast Blog Markdown Support

To make this work, create a new folder: /app/blog/_assets/articles

Create and edit your markdown (.md) articles here.

Also, in the /app/blog/_assets/content.js file, remove the `content` key from the articles object.

### Install libraries

```
npm install gray-matter remark remark-rehype rehype-react rehype-raw rehype-stringify html-react-parser
```

### Create new file: libs/markdown.js

```javascript
import fs from "fs";
import path from "path";
import matter from "gray-matter";
import { unified } from "unified";
import parse from "remark-parse";
import remark2rehype from "remark-rehype";
import raw from "rehype-raw";
import stringify from "rehype-stringify";

const postsDirectory = path.join(process.cwd(), "app/blog/_assets/articles");

export const getPostContent = (id) => {
  const fullPath = path.join(postsDirectory, `${id}.md`);
  const fileContents = fs.readFileSync(fullPath, "utf8");
  const { data, content } = matter(fileContents);
  const htmlContent = renderMarkdown(content);
  return {
    id,
    ...data,
    content: htmlContent,
  };
};

const renderMarkdown = (markdown) => {
  const processor = unified()
    .use(parse)
    .use(remark2rehype, { allowDangerousHtml: true })
    .use(raw)
    .use(stringify);

  const result = processor.processSync(markdown);
  return result.toString();
};

```

### Create new file: app/blog/_assets/components/MarkdownContent.js

```javascript
import React from "react";
import parse from "html-react-parser";

const styles = {
  h2: "text-2xl lg:text-4xl font-bold tracking-tight mb-4 text-base-content my-10",
  h3: "text-xl lg:text-2xl font-bold tracking-tight mb-2 text-base-content my-8",
  p: "text-base-content/90 leading-relaxed py-6",
  ul: "list-inside list-disc text-base-content/90 leading-relaxed my-6",
  ol: "my-6",
  li: "list-item my-2",
  code: "text-sm font-mono bg-neutral text-neutral-content p-6 rounded-box my-4 overflow-x-scroll select-all",
  codeInline:
    "text-sm font-mono bg-base-300 px-1 py-0.5 rounded-box select-all",
};

const applyCustomClasses = (node) => {
  if (node.type === "tag") {
    let className = node.attribs.class || "";

    className = styles[node.name];

    node.attribs.class = className;
  }
};

export const MarkdownContent = ({ htmlString }) => {
  const options = {
    replace: (domNode) => {
      applyCustomClasses(domNode);
      return domNode;
    },
  };

  return <>{parse(htmlString, options)}</>;
};
```

### Edit app/blog/[articleId]/page.js

```javascript
import { getPostContent } from "@/libs/markdown";
import { MarkdownContent } from "../_assets/components/MarkdownContent";

export default async function Article({ params }) {
  const article = articles.find((article) => article.slug === params.articleId);
  const { content } = getPostContent(article.slug);
   // continue with existing code
   
  return (
  	<div>
    {/* existing jsx */}
     {/* ARTICLE CONTENT */}
     <section className="w-full max-md:pt-4 md:pr-20 ">
        <MarkdownContent htmlString={content} /
     </section>
    </div>
  )
```
