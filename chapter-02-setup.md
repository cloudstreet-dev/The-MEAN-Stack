# Chapter 2: Setting Up Your Development Environment

## The Tools of the Trade

Before you can build anything, you need the right tools. This chapter is like assembling your lightsaber before becoming a Jedi—not glamorous, but absolutely essential. We'll install everything you need to be productive with the MEAN stack.

Fair warning: this chapter involves downloading things, running installers, and configuring your computer. It's about as exciting as assembling IKEA furniture, but at least you won't have any mysterious extra screws at the end.

## The Shopping List

Here's what we'll install:

1. **Node.js and npm** - The JavaScript runtime and package manager
2. **MongoDB** - Your database
3. **Git** - Version control (you'll thank me later)
4. **A code editor** - Your primary workspace
5. **Postman** or similar - For testing APIs
6. **Angular CLI** - Command-line tools for Angular

Let's tackle these one by one.

## Installing Node.js and npm

Node.js is the foundation of your MEAN stack. When you install Node.js, npm (Node Package Manager) comes bundled with it, which is convenient because you'll use npm constantly.

### For macOS

**Option 1: Official Installer (Easiest)**

1. Visit [nodejs.org](https://nodejs.org/)
2. Download the LTS (Long Term Support) version—it's the stable one
3. Run the installer
4. Click "Next" repeatedly while contemplating life choices
5. Celebrate mildly when it finishes

**Option 2: Homebrew (Cooler)**

If you have Homebrew installed (and if you don't, you should—it's the missing package manager for macOS):

```bash
brew install node
```

That's it. Homebrew is like magic, except real.

### For Windows

**Option 1: Official Installer**

1. Visit [nodejs.org](https://nodejs.org/)
2. Download the LTS version for Windows
3. Run the installer
4. Accept the license agreement (after reading it thoroughly, of course)
5. Make sure "Add to PATH" is checked—this is important
6. Let it install Chocolatey tools if prompted (they're useful)
7. Click "Install" and wait

**Option 2: Windows Package Manager (winget)**

If you're on Windows 10/11 with winget:

```bash
winget install OpenJS.NodeJS.LTS
```

**Option 3: Chocolatey**

If you have Chocolatey installed:

```bash
choco install nodejs-lts
```

### For Linux

**Ubuntu/Debian:**

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**Fedora:**

```bash
sudo dnf install nodejs
```

**For other distros:**

You know what you're doing. You're using Linux. You've probably compiled Node.js from source just for fun.

### Verifying the Installation

Open a new terminal (important: NEW terminal—the old one doesn't know about the changes) and run:

```bash
node --version
npm --version
```

You should see version numbers. If you see "command not found," something went wrong. Try:
- Restarting your terminal
- Restarting your computer (the classic IT solution)
- Googling the error message (the modern IT solution)

**Recommended Versions:**
- Node.js: v18.x or higher (v20.x is great too)
- npm: v9.x or higher (usually comes with Node.js)

## Optional but Recommended: nvm (Node Version Manager)

Here's a pro tip: install `nvm` (Node Version Manager). It lets you switch between Node.js versions easily, which is useful when different projects require different versions.

### Installing nvm on macOS/Linux

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

Then restart your terminal and install Node.js via nvm:

```bash
nvm install --lts
nvm use --lts
```

### Installing nvm on Windows

Windows has its own version called `nvm-windows`:

1. Download the installer from [github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases)
2. Run the installer
3. Open a new terminal and run:

```bash
nvm install lts
nvm use lts
```

With nvm, you can easily switch between Node versions:

```bash
nvm install 18
nvm use 18
# Later...
nvm install 20
nvm use 20
```

It's like having multiple Node.js installations that don't fight with each other.

## Installing MongoDB

MongoDB is your database. There are several ways to run it: locally, via Docker, or in the cloud. Let's cover all options so you can choose your own adventure.

### Option 1: Local Installation

**macOS:**

```bash
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

**Windows:**

1. Visit [mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)
2. Download MongoDB Community Server
3. Run the installer
4. Choose "Complete" installation
5. Install MongoDB as a Windows Service (check the box)
6. Optionally install MongoDB Compass (the GUI tool—it's actually useful)

**Linux (Ubuntu/Debian):**

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -sc)/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
```

### Option 2: Docker (Recommended for Development)

If you have Docker installed (and Docker is genuinely fantastic for development):

```bash
docker pull mongo
docker run -d -p 27017:27017 --name mongodb mongo
```

This creates a MongoDB container that runs in the background. To stop it:

```bash
docker stop mongodb
```

To start it again:

```bash
docker start mongodb
```

To completely remove it:

```bash
docker rm mongodb
```

Docker keeps your system clean and lets you start fresh whenever you need to.

### Option 3: MongoDB Atlas (Cloud)

Don't want to install anything locally? MongoDB Atlas offers a free tier that's perfect for learning:

1. Visit [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas)
2. Sign up for a free account
3. Create a free M0 cluster (it's actually free, not "free for 30 days")
4. Set up a database user and password
5. Whitelist your IP address (or use 0.0.0.0/0 for anywhere, though that's less secure)
6. Get your connection string

You'll use this connection string in your Node.js applications. It looks like:

```
mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/<dbname>
```

Atlas is great because:
- Nothing to install
- Automatic backups
- Easy to share with teammates
- Accessible from anywhere
- Free tier is generous

### Verifying MongoDB Installation

If you installed locally or via Docker, verify it's running:

```bash
mongosh
```

You should see the MongoDB shell. If it connects, you're good. Type `exit` to leave.

If `mongosh` isn't found, you might need to install the MongoDB Shell separately:

```bash
# macOS
brew install mongosh

# Windows
winget install MongoDB.Shell

# Linux
sudo apt-get install -y mongodb-mongosh
```

## Installing Git

You might already have Git installed. Let's check:

```bash
git --version
```

If you see a version number, you're done with this section. Go get a coffee.

If not:

**macOS:**

```bash
brew install git
```

Or download from [git-scm.com](https://git-scm.com/)

**Windows:**

Download Git for Windows from [git-scm.com](https://git-scm.com/). During installation:
- Use default editor (or pick VS Code if you have it)
- Choose "Git from the command line and also from 3rd-party software"
- Use the OpenSSL library
- Checkout Windows-style, commit Unix-style line endings
- Use MinTTY (the default terminal)
- Enable Git Credential Manager

**Linux:**

```bash
# Ubuntu/Debian
sudo apt-get install git

# Fedora
sudo dnf install git
```

### Configure Git

Set up your identity (Git will nag you otherwise):

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

These show up in your commit history, so use something professional if you plan to share code publicly. "xXxDarkKnight420xXx" is not professional, no matter how cool you think it sounds.

## Choosing and Configuring a Code Editor

You need a good code editor. Not Notepad. Not Word. A real code editor.

### Visual Studio Code (Recommended)

VS Code is free, powerful, and has extensions for everything. It's the Swiss Army chainsaw of code editors.

**Installation:**

- **macOS:** `brew install --cask visual-studio-code` or download from [code.visualstudio.com](https://code.visualstudio.com/)
- **Windows:** Download from [code.visualstudio.com](https://code.visualstudio.com/) or `winget install Microsoft.VisualStudioCode`
- **Linux:** Use your package manager or download from the website

**Essential Extensions:**

Open VS Code, press `Ctrl+Shift+X` (or `Cmd+Shift+X` on Mac), and install:

1. **ESLint** - Catches JavaScript errors before you run your code
2. **Prettier** - Formats your code automatically
3. **Angular Language Service** - Angular autocomplete and error checking
4. **MongoDB for VS Code** - View and query databases from VS Code
5. **Thunder Client** or **REST Client** - Test APIs without leaving the editor
6. **GitLens** - Supercharges Git integration
7. **Auto Rename Tag** - Renames closing HTML tags automatically
8. **Bracket Pair Colorizer** (or use built-in bracket pair colorization)
9. **Path Intellisense** - Autocomplete file paths
10. **Material Icon Theme** or **Material Theme** - Make VS Code pretty

**Recommended Settings:**

Press `Ctrl+,` (or `Cmd+,` on Mac) to open settings, then click the `{}` icon for JSON settings. Add:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.wordWrap": "on",
  "files.autoSave": "onFocusChange",
  "javascript.updateImportsOnFileMove.enabled": "always",
  "typescript.updateImportsOnFileMove.enabled": "always",
  "editor.bracketPairColorization.enabled": true,
  "editor.guides.bracketPairs": true
}
```

### Alternative Editors

**WebStorm:** Powerful, feature-rich, made by JetBrains. It's paid, but there's a free trial and student licenses available. Some developers swear by it. It does everything but make your coffee.

**Sublime Text:** Fast, lightweight, minimalist. It's technically paid, but the trial never expires (though you should buy it if you use it regularly).

**Atom:** GitHub's editor. It's being sunset by GitHub in favor of VS Code, but it still works fine.

**Vim/Neovim:** For the brave souls who want to configure everything themselves. If you're asking "why?", vim probably isn't for you. If you're asking "how?", you're probably already using it.

## Installing Postman (or Alternatives)

You'll need a tool to test your APIs while building them. Postman is the most popular choice.

**Installing Postman:**

1. Visit [postman.com](https://www.postman.com/)
2. Download and install
3. Create a free account (optional but recommended—syncs your work)

**Alternatives:**

- **Insomnia:** Similar to Postman, some find it cleaner
- **Thunder Client:** VS Code extension (lightweight, built-in)
- **cURL:** Command-line tool (for masochists and command-line purists)
- **HTTPie:** Better cURL (still command-line, but prettier)

**Using cURL (if you want to feel like a hacker):**

cURL comes pre-installed on macOS and Linux. For Windows, it's included in Windows 10+.

```bash
# GET request
curl http://localhost:3000/api/tasks

# POST request
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn cURL","completed":false}'
```

Use Postman for development and cURL when you want to feel cool.

## Installing Angular CLI

The Angular CLI (Command Line Interface) is essential for Angular development. Install it globally via npm:

```bash
npm install -g @angular/cli
```

The `-g` flag installs it globally, making the `ng` command available everywhere.

Verify installation:

```bash
ng version
```

You should see Angular CLI version and other info. If you see an error about permissions on macOS/Linux, you might need to fix npm permissions:

```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

Then try installing again.

On Windows, if you get a script execution error, run PowerShell as Administrator and execute:

```bash
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

## Optional: Installing TypeScript Globally

While Angular includes TypeScript, having it globally can be useful:

```bash
npm install -g typescript
```

Verify:

```bash
tsc --version
```

TypeScript is like JavaScript with training wheels—or perhaps JavaScript with a safety harness. It catches type-related errors before they become runtime disasters.

## Setting Up a Project Directory

Let's create a workspace for your projects:

```bash
# macOS/Linux
mkdir ~/mean-projects
cd ~/mean-projects

# Windows
mkdir %USERPROFILE%\mean-projects
cd %USERPROFILE%\mean-projects
```

This gives you a dedicated place for your MEAN projects. Organization matters when you're juggling multiple projects (or when you try to remember what you built six months ago).

## Creating a Test Project

Let's verify everything works by creating a tiny test project.

```bash
mkdir test-mean-setup
cd test-mean-setup
npm init -y
```

This creates a `package.json` file. Now install Express.js:

```bash
npm install express
```

Create a file called `test.js`:

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello MEAN Stack!',
    node: process.version,
    express: require('express/package.json').version
  });
});

app.listen(port, () => {
  console.log(`Test server running at http://localhost:${port}`);
});
```

Run it:

```bash
node test.js
```

Open your browser and visit `http://localhost:3000`. You should see JSON output with a message and version numbers.

If it works, congratulations! Your Node.js and npm are properly configured.

Press `Ctrl+C` in the terminal to stop the server.

## Testing MongoDB Connection

Let's verify MongoDB works. Create a file called `test-mongo.js`:

```javascript
const { MongoClient } = require('mongodb');

// Use this URL for local MongoDB
const url = 'mongodb://localhost:27017';

// Or use your Atlas connection string if using cloud
// const url = 'mongodb+srv://username:password@cluster.xxxxx.mongodb.net/test';

const client = new MongoClient(url);

async function testConnection() {
  try {
    await client.connect();
    console.log('Connected successfully to MongoDB!');

    const db = client.db('testdb');
    const collection = db.collection('testcollection');

    // Insert a test document
    const result = await collection.insertOne({ test: 'Hello MongoDB!' });
    console.log('Inserted document with _id:', result.insertedId);

    // Read it back
    const doc = await collection.findOne({ test: 'Hello MongoDB!' });
    console.log('Found document:', doc);

    // Clean up
    await collection.deleteOne({ _id: result.insertedId });
    console.log('Test document deleted');

  } catch (error) {
    console.error('MongoDB connection error:', error);
  } finally {
    await client.close();
  }
}

testConnection();
```

But first, install the MongoDB driver:

```bash
npm install mongodb
```

Run the test:

```bash
node test-mongo.js
```

If you see success messages, MongoDB is working correctly!

## Testing Angular CLI

Create a tiny Angular app to verify the CLI works:

```bash
ng new test-angular --minimal --skip-git
```

This will ask a few questions:
- Routing? Say No for this test
- Stylesheet format? Pick CSS

It'll take a minute to create the project and install dependencies. When it's done:

```bash
cd test-angular
ng serve
```

Visit `http://localhost:4200` in your browser. You should see the Angular welcome page.

Press `Ctrl+C` to stop the dev server.

## Cleaning Up

Those test projects served their purpose. You can delete them:

```bash
cd ..
rm -rf test-mean-setup test-angular

# On Windows use:
# rmdir /s test-mean-setup
# rmdir /s test-angular
```

## Troubleshooting Common Issues

### "Command not found" errors

- Make sure you opened a NEW terminal after installation
- Verify the software is in your PATH
- On Windows, restart your computer (seriously, Windows sometimes needs this)

### Port already in use

If you see "Port 3000 is already in use":

```bash
# macOS/Linux - Find what's using the port
lsof -i :3000

# Then kill it (use the PID from above)
kill -9 <PID>

# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

Or just use a different port in your code.

### MongoDB won't start

- Check if it's already running: `mongosh` should connect if it is
- On macOS: `brew services restart mongodb-community`
- On Linux: `sudo systemctl restart mongod`
- On Windows: Check Services (search in Start menu) and restart MongoDB service
- Using Docker: `docker restart mongodb`

### npm permission errors (macOS/Linux)

Never use `sudo` with npm! Instead, fix the permissions:

```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

Then try your npm command again.

### Angular CLI won't run (Windows)

PowerShell script execution policy might be blocking it:

```bash
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Run PowerShell as Administrator and execute that command.

## Useful Terminal Commands

Here are commands you'll use constantly:

```bash
# Node.js and npm
node script.js          # Run a JavaScript file
npm install package     # Install a package locally
npm install -g package  # Install globally
npm start              # Run the start script from package.json
npm test               # Run tests
npm run <script>       # Run a custom script

# Angular CLI
ng new project-name    # Create new Angular project
ng serve              # Start development server
ng generate component name  # Generate a component
ng build              # Build for production

# MongoDB
mongosh               # Open MongoDB shell
mongosh "connection-string"  # Connect to specific database

# Git
git init              # Initialize a repository
git add .             # Stage all changes
git commit -m "msg"   # Commit with message
git status            # Check status
git log               # View commit history

# General terminal
pwd                   # Print working directory
ls                    # List files (macOS/Linux)
dir                   # List files (Windows)
cd directory          # Change directory
mkdir directory       # Create directory
rm -rf directory      # Remove directory (be careful!)
```

## Setting Up a Development Workflow

Here's a recommended workflow setup:

1. **Terminal/iTerm2/Windows Terminal** - Your command center
2. **VS Code** - Your primary editor
3. **Browser with DevTools** - Chrome or Firefox Developer Edition
4. **Postman** - For API testing
5. **MongoDB Compass** - For viewing database contents

**Pro Tip:** Use multiple monitors if possible. One for code, one for browser/documentation, one for terminal. If you only have one monitor, learn to use virtual desktops (Spaces on macOS, Task View on Windows, workspaces on Linux).

## Optional: Installing Useful Tools

### nodemon

Automatically restarts your Node.js server when files change:

```bash
npm install -g nodemon
```

Use `nodemon server.js` instead of `node server.js`.

### http-server

Quick static file server:

```bash
npm install -g http-server
```

Run `http-server` in any directory to serve static files.

### JSON Server

Mock REST API for testing:

```bash
npm install -g json-server
```

Create a `db.json` file and run `json-server --watch db.json` for an instant REST API.

### MongoDB Compass

GUI for MongoDB (if you didn't install it earlier):

Download from [mongodb.com/try/download/compass](https://www.mongodb.com/try/download/compass)

It's actually quite good for visualizing your data and running queries without touching the command line.

## Your Development Environment Checklist

Before moving to the next chapter, make sure you have:

- [ ] Node.js and npm installed and working
- [ ] MongoDB running (locally, Docker, or Atlas account set up)
- [ ] Git installed and configured
- [ ] Code editor (VS Code recommended) with extensions
- [ ] Postman or alternative API client
- [ ] Angular CLI installed globally
- [ ] Successfully created and ran test projects
- [ ] Terminal configured and comfortable to use

If you checked all those boxes, you're ready to start building.

## What's Next?

Now that your development environment is set up, we can start writing actual code. In the next chapter, we'll dive into Node.js fundamentals—understanding the runtime that powers your backend, working with modules, handling asynchronous code, and building your first real Node.js application.

The boring setup work is done. The fun part begins now.

---

*"Setting up a development environment is like meal prep: tedious and time-consuming, but your future self will thank you."* — Developer Who Learned the Hard Way
