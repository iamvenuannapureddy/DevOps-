<h1>Project-9</h1>

### **Project: Version-Controlled Documentation or Blog Using GitHub Pages**

The goal of this project is to create, maintain, and version-control a technical blog or documentation site using tools like **Jekyll**, **Hugo**, or **Docusaurus**. The project will use **GitHub Pages** for hosting and **branches/PRs** for collaborative writing and content review.

---

### **1. Project Directory Structure**

#### Example using **Jekyll**:

```bash
documentation-blog/
│
├── _config.yml                        # Jekyll configuration file
├── _posts/                            # Blog posts or documentation content
│   └── 2024-10-01-welcome-to-my-blog.md
│
├── _layouts/                          # Layout templates for pages
│   └── default.html
│
├── _includes/                         # HTML partials included in layouts
│   └── header.html
│   └── footer.html
│
├── _sass/                             # Optional SASS files for custom styles
│   └── custom.scss
│
├── assets/                            # CSS, JavaScript, and image files
│   └── main.css
│
├── index.html                         # Home page content
├── LICENSE                            # License for open-source sharing
└── README.md                          # Project documentation
```

---

### **2. Project Files Content**

#### **2.1. _config.yml**

Jekyll's configuration file that defines site settings:

```yaml
title: "My Tech Blog"
description: "A blog about my technical adventures."
baseurl: ""
url: "https://yourusername.github.io"
theme: minima
markdown: kramdown
paginate: 5
```

#### **2.2. _posts/2024-10-01-welcome-to-my-blog.md**

An example blog post:

```markdown
---
layout: post
title:  "Welcome to My Blog!"
date:   2024-10-01 12:00:00
categories: jekyll blog tech
---

This is my first blog post! In this blog, I will share my thoughts on technology, coding, DevOps, and more. Stay tuned for exciting content!
```

#### **2.3. _layouts/default.html**

The default layout template for the blog:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>{{ page.title }}</title>
  <link rel="stylesheet" href="{{ "/assets/main.css" | relative_url }}">
</head>
<body>
  <header>
    {% include header.html %}
  </header>
  
  <main>
    {{ content }}
  </main>
  
  <footer>
    {% include footer.html %}
  </footer>
</body>
</html>
```

#### **2.4. assets/main.css**

Custom styles for the blog:

```css
body {
  font-family: Arial, sans-serif;
  color: #333;
}

header, footer {
  background-color: #f8f8f8;
  padding: 10px 0;
  text-align: center;
}
```

#### **2.5. README.md**

Project documentation to guide contributors and users:

```markdown
# My Tech Blog

## Overview

This project hosts a personal blog created with Jekyll and hosted on GitHub Pages. The blog covers various technical topics like coding, DevOps, and open-source contributions.

## Getting Started

1. Clone the repository:

```bash
git clone https://github.com/yourusername/documentation-blog.git
cd documentation-blog
```

2. Install Jekyll and its dependencies:

```bash
bundle install
```

3. Serve the site locally:

```bash
bundle exec jekyll serve
```

4. Open `http://localhost:4000` in your browser to view the blog.

## Contributing

1. Create a new branch for your changes:

```bash
git checkout -b feature/new-post
```

2. Write a new blog post in the `_posts/` directory.
3. Commit and push your changes:

```bash
git add .
git commit -m "Add new blog post"
git push origin feature/new-post
```

4. Open a pull request (PR) on GitHub for review.

## License

This project is licensed under the MIT License.
```

---

### **3. Steps to Create the Documentation or Blog**

#### **Step 1: Install Jekyll (for Jekyll blogs)**

1. Install Ruby and Bundler.
2. Install Jekyll:

   ```bash
   gem install jekyll bundler
   ```

#### **Step 2: Set Up a GitHub Repository**

1. Create a new GitHub repository (e.g., `documentation-blog`).
2. Clone the repository locally:

   ```bash
   git clone https://github.com/yourusername/documentation-blog.git
   cd documentation-blog
   ```

#### **Step 3: Initialize Jekyll**

1. Create a new Jekyll site:

   ```bash
   jekyll new .
   ```

2. Add content to `_posts/`, customize `_layouts/`, `_includes/`, and add your CSS files in `assets/`.

#### **Step 4: Commit and Push the Changes**

1. Add and commit your changes:

   ```bash
   git add .
   git commit -m "Initial setup of blog"
   git push -u origin main
   ```

#### **Step 5: Set Up GitHub Pages**

1. Go to your repository’s **Settings** in GitHub.
2. Scroll down to **GitHub Pages**.
3. In the **Source** section, select the **main** branch and click **Save**.
4. Your site will be published at `https://yourusername.github.io/documentation-blog`.

#### **Step 6: Write and Publish Content**

1. Use the `_posts/` directory to add new posts or documentation pages.
2. Write each post using markdown, following the date-based naming convention (`YYYY-MM-DD-title.md`).

#### **Step 7: Collaborate Using Git and Pull Requests**

1. Use branches for new features or blog posts:

   ```bash
   git checkout -b feature/new-blog-post
   ```

2. Push the new branch:

   ```bash
   git push origin feature/new-blog-post
   ```

3. Open a pull request (PR) on GitHub for review and collaboration.

---

### **4. GitHub Actions for Auto-Deployment**

To automate the deployment of your Jekyll site, you can set up **GitHub Actions** to build and deploy the site whenever changes are pushed to the main branch.

#### **4.1. .github/workflows/jekyll.yml**

A GitHub Actions workflow for deploying Jekyll:

```yaml
name: Jekyll Build & Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: '2.7'

    - name: Install dependencies
      run: |
        bundle install

    - name: Build the site
      run: |
        bundle exec jekyll build

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./_site
```

---

### **5. Hosting on Custom Domain (Optional)**

1. In your repository, create a file called `CNAME` and add your custom domain (e.g., `myblog.com`).
2. Update your domain’s DNS settings to point to GitHub Pages.

---

### **6. Tools for Blog Creation**

- **Jekyll**: A static site generator built for blogs.
- **Hugo**: Another static site generator that's fast and flexible.
- **Docusaurus**: Ideal for versioned technical documentation.
- **GitHub Pages**: Free hosting for GitHub repositories.

This setup gives you an end-to-end version-controlled blog or documentation site that supports collaboration, continuous deployment, and easy content management through GitHub. Let me know if you need any further details!
