<h1>Project-1</h1>
Creating a **Personal Portfolio Website** using **GitHub Pages** is a great way to showcase your projects and experience. Here’s a complete **end-to-end guide** to building, hosting, and deploying your portfolio website using **HTML/CSS/JavaScript** and **GitHub Pages**.

### **Steps to Create a Personal Portfolio Website**

---

### **1. Create the Website Locally**

Before pushing your website to GitHub, let’s first create the website’s structure on your local machine.

#### **Step 1.1: Set up Project Directory**

Create a folder for your portfolio website:
```bash
mkdir my-portfolio
cd my-portfolio
```

Inside this folder, create the necessary files and folders for the website:

```bash
.
├── index.html
├── css
│   └── style.css
└── js
    └── script.js
```

- `index.html`: The main HTML file for your portfolio.
- `css/style.css`: The CSS file for styling the website.
- `js/script.js`: (Optional) The JavaScript file for dynamic behavior.

#### **Step 1.2: Create a Basic HTML Structure**

Here’s a simple structure for your `index.html` file:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Your Name - Portfolio</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <header>
        <h1>Welcome to My Portfolio</h1>
        <nav>
            <ul>
                <li><a href="#about">About</a></li>
                <li><a href="#projects">Projects</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <section id="about">
        <h2>About Me</h2>
        <p>Hello! I'm [Your Name], a passionate developer. Here's a bit about me.</p>
    </section>

    <section id="projects">
        <h2>My Projects</h2>
        <div class="project">
            <h3>Project 1</h3>
            <p>Project description goes here.</p>
        </div>
        <div class="project">
            <h3>Project 2</h3>
            <p>Project description goes here.</p>
        </div>
    </section>

    <section id="contact">
        <h2>Contact Me</h2>
        <p>Email: yourname@example.com</p>
    </section>

    <footer>
        <p>&copy; 2024 Your Name</p>
    </footer>
    
    <script src="js/script.js"></script>
</body>
</html>
```

#### **Step 1.3: Add Some Basic CSS**

In the `css/style.css` file, add styles to improve the appearance of your website:

```css
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
}

header {
    background-color: #333;
    color: #fff;
    padding: 20px;
    text-align: center;
}

nav ul {
    list-style-type: none;
    padding: 0;
}

nav ul li {
    display: inline;
    margin: 0 15px;
}

nav ul li a {
    color: #fff;
    text-decoration: none;
}

section {
    padding: 20px;
    margin: 20px 0;
}

footer {
    text-align: center;
    padding: 10px;
    background-color: #333;
    color: #fff;
}
```

---

### **2. Create a GitHub Repository**

#### **Step 2.1: Initialize a Git Repository Locally**

In your project directory, initialize a Git repository and commit the code:

```bash
git init
git add .
git commit -m "Initial commit - Personal Portfolio Website"
```

#### **Step 2.2: Create a GitHub Repository**

1. Go to [GitHub](https://github.com) and log into your account.
2. Click on the **New repository** button.
3. Name the repository something like `my-portfolio` (it can be anything, but if you use `username.github.io`, GitHub will use it as your website's URL).
4. Set the repository to **public** and click **Create repository**.

#### **Step 2.3: Push the Website Code to GitHub**

After creating the GitHub repository, push your local code to GitHub:

```bash
git remote add origin https://github.com/your-username/my-portfolio.git
git branch -M main
git push -u origin main
```

---

### **3. Enable GitHub Pages**

Now that your website code is on GitHub, you need to enable GitHub Pages to host the site.

#### **Step 3.1: Enable GitHub Pages**

1. Go to your repository on GitHub.
2. Click on the **Settings** tab.
3. Scroll down to the **Pages** section (usually on the left-hand side or at the bottom).
4. In the **Source** section, select the `main` branch (or another branch you want to use).
5. Optionally, select a specific folder, such as `/root` (this is where your `index.html` file is located).
6. Click **Save**.

#### **Step 3.2: Access Your Website**

After enabling GitHub Pages, GitHub will provide a URL where your site is hosted. It will be something like:

```
https://your-username.github.io/my-portfolio/
```

It may take a few minutes for your site to be live.

---

### **4. (Optional) Use a Custom Domain**

If you have a custom domain name (e.g., `www.mywebsite.com`), you can configure GitHub Pages to use that domain.

#### **Step 4.1: Set Up Custom Domain in GitHub**

1. In the **Pages** section of your repository settings, there’s an option to add a custom domain.
2. Enter your domain name (e.g., `www.mywebsite.com`).

#### **Step 4.2: Configure Your Domain’s DNS Settings**

To link your custom domain to GitHub Pages, you’ll need to add the following DNS records through your domain registrar:

- **A records**:
   - Points to these IP addresses:
     ```
     185.199.108.153
     185.199.109.153
     185.199.110.153
     185.199.111.153
     ```

- **CNAME record**:
   - If you’re using a subdomain (like `www`), add a CNAME record pointing to `your-username.github.io`.

---

### **5. Continuous Updates**

Whenever you make changes to your website code (e.g., updating your portfolio, adding projects), follow these steps to push updates to GitHub Pages:

1. **Make changes** to your local code (HTML, CSS, or JS files).
2. **Stage and commit** the changes:
   ```bash
   git add .
   git commit -m "Updated portfolio content"
   ```
3. **Push the changes** to GitHub:
   ```bash
   git push origin main
   ```

The changes will automatically be reflected on your GitHub Pages site.

---

### **Summary of Steps**

1. **Set up a basic portfolio website** using HTML/CSS/JavaScript on your local machine.
2. **Create a GitHub repository** to host the website’s code.
3. **Push the website code** to the GitHub repository.
4. **Enable GitHub Pages** to deploy the website.
5. Optionally, **configure a custom domain** for your portfolio.

Once set up, your portfolio will be accessible online, and you can continuously update it by pushing new changes to your GitHub repository. Let me know if you need any additional help with the deployment or design!
