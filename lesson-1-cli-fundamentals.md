# Lesson 1: CLI Fundamentals with Bun.js

Learn to build command-line interfaces step by step, starting from the absolute basics.

## Prerequisites
- Bun.js installed (`curl -fsSL https://bun.sh/install | bash`)
- Basic TypeScript knowledge

## Step 1: Understanding Command Line Arguments

When you run a command like `git add file.txt`, here's what happens:
- `git` is the program
- `add` and `file.txt` are **arguments** passed to the program
- The program can read these arguments and decide what to do

Let's see this in action. Create your first CLI:

```typescript
#!/usr/bin/env bun
// step1.ts

console.log("Arguments received:", process.argv);
```

Run it and see what happens:
```bash
chmod +x step1.ts
./step1.ts
./step1.ts hello world
```

**What you'll see:** `process.argv` contains ALL arguments, including the path to bun and your script.

## Step 2: Getting Just Your Arguments

The first two items in `process.argv` are always the runtime and script path. Your actual arguments start at index 2:

```typescript
#!/usr/bin/env bun
// step2.ts

const args = process.argv.slice(2);
console.log("Your arguments:", args);
console.log("Number of arguments:", args.length);
```

Try it:
```bash
./step2.ts
./step2.ts hello
./step2.ts hello world 123
```

## Step 3: Using Your First Argument

Now let's do something useful with the first argument:

```typescript
#!/usr/bin/env bun
// step3.ts

const args = process.argv.slice(2);
const name = args[0];

if (name) {
    console.log(`Hello, ${name}!`);
} else {
    console.log("Hello, World!");
}
```

Test it:
```bash
./step3.ts Alice
./step3.ts
```

## Step 4: Handling Multiple Arguments

Let's expand to handle more arguments:

```typescript
#!/usr/bin/env bun
// step4.ts

const args = process.argv.slice(2);

if (args.length === 0) {
    console.log("Hello, World!");
} else if (args.length === 1) {
    console.log(`Hello, ${args[0]}!`);
} else {
    console.log(`Hello, ${args.join(" and ")}!`);
}
```

Try:
```bash
./step4.ts Alice
./step4.ts Alice Bob
./step4.ts Alice Bob Charlie
```

## Step 5: Understanding Argument Patterns

Real CLI tools use patterns like `--name Alice` or `-u`. Let's learn to recognize these:

```typescript
#!/usr/bin/env bun
// step5.ts

const args = process.argv.slice(2);

console.log("All arguments:", args);

// Look for different patterns
args.forEach((arg, index) => {
    if (arg.startsWith('--')) {
        console.log(`Found long flag: ${arg}`);
    } else if (arg.startsWith('-')) {
        console.log(`Found short flag: ${arg}`);
    } else {
        console.log(`Found value: ${arg}`);
    }
});
```

Try different argument styles:
```bash
./step5.ts --name Alice -u hello
./step5.ts --verbose --output=file.txt
```

## Step 6: Parsing Simple Flags

Let's handle one flag at a time. Start with a boolean flag:

```typescript
#!/usr/bin/env bun
// step6.ts

const args = process.argv.slice(2);

// Check if --loud flag exists
const isLoud = args.includes('--loud');
const name = args.find(arg => !arg.startsWith('-')) || 'World';

let message = `Hello, ${name}!`;
if (isLoud) {
    message = message.toUpperCase();
}

console.log(message);
```

Test it:
```bash
./step6.ts Alice
./step6.ts Alice --loud
./step6.ts --loud Bob
```

## Step 7: Parsing Named Arguments

Now let's handle `--name=Alice` style arguments:

```typescript
#!/usr/bin/env bun
// step7.ts

const args = process.argv.slice(2);

// Find --name argument
let name = 'World';
const nameArg = args.find(arg => arg.startsWith('--name='));
if (nameArg) {
    name = nameArg.split('=')[1];
}

// Check for --loud flag
const isLoud = args.includes('--loud');

let message = `Hello, ${name}!`;
if (isLoud) {
    message = message.toUpperCase();
}

console.log(message);
```

Try:
```bash
./step7.ts --name=Alice
./step7.ts --name=Bob --loud
```

## Step 8: Adding Basic Validation

Let's add simple validation to make sure we get what we expect:

```typescript
#!/usr/bin/env bun
// step8.ts

const args = process.argv.slice(2);

// We need at least a name
if (args.length === 0) {
    console.error("Error: Please provide a name");
    console.log("Usage: step8.ts <name> [--loud]");
    process.exit(1);
}

const name = args[0];
const isLoud = args.includes('--loud');

// Validate name isn't empty
if (name.trim() === '') {
    console.error("Error: Name cannot be empty");
    process.exit(1);
}

let message = `Hello, ${name}!`;
if (isLoud) {
    message = message.toUpperCase();
}

console.log(message);
```

Test the validation:
```bash
./step8.ts
./step8.ts ""
./step8.ts Alice
./step8.ts Alice --loud
```

## Step 9: Better Error Handling

Let's make error handling more user-friendly:

```typescript
#!/usr/bin/env bun
// step9.ts

const args = process.argv.slice(2);

const showUsage = () => {
    console.log("Usage: step9.ts <name> [--loud]");
    console.log("  name    Your name to greet");
    console.log("  --loud  Make the greeting uppercase");
};

// Check if user asked for help
if (args.includes('--help') || args.includes('-h')) {
    showUsage();
    process.exit(0);
}

// Validate we have a name
if (args.length === 0) {
    console.error("❌ Error: Please provide a name");
    showUsage();
    process.exit(1);
}

const name = args[0];
const isLoud = args.includes('--loud');

// Validate name isn't empty
if (name.trim() === '') {
    console.error("❌ Error: Name cannot be empty");
    process.exit(1);
}

let message = `Hello, ${name}!`;
if (isLoud) {
    message = message.toUpperCase();
}

console.log(message);
```

Try the help:
```bash
./step9.ts --help
./step9.ts -h
```

## Step 10: Reading Your First File

Let's read a simple text file. First, create a test file:

```bash
echo "Hello from file!" > test.txt
```

Now read it:

```typescript
#!/usr/bin/env bun
// step10.ts

import { readFileSync } from 'fs';

const args = process.argv.slice(2);

if (args.length === 0) {
    console.error("❌ Please provide a filename");
    console.log("Usage: step10.ts <filename>");
    process.exit(1);
}

const filename = args[0];

try {
    const content = readFileSync(filename, 'utf-8');
    console.log("📄 File contents:");
    console.log(content);
} catch (error) {
    console.error(`❌ Error reading file: ${error.message}`);
    process.exit(1);
}
```

Test it:
```bash
./step10.ts test.txt
./step10.ts nonexistent.txt
```

## Step 11: Checking if Files Exist

Let's be more careful about files that might not exist:

```typescript
#!/usr/bin/env bun
// step11.ts

import { readFileSync, existsSync } from 'fs';

const args = process.argv.slice(2);

if (args.length === 0) {
    console.error("❌ Please provide a filename");
    process.exit(1);
}

const filename = args[0];

// Check if file exists first
if (!existsSync(filename)) {
    console.error(`❌ File not found: ${filename}`);
    process.exit(1);
}

try {
    const content = readFileSync(filename, 'utf-8');
    console.log(`📄 Reading: ${filename}`);
    console.log(content);
} catch (error) {
    console.error(`❌ Error reading file: ${error.message}`);
    process.exit(1);
}
```

## Step 12: Writing to Files

Let's write some content to a file:

```typescript
#!/usr/bin/env bun
// step12.ts

import { writeFileSync } from 'fs';

const args = process.argv.slice(2);

if (args.length < 2) {
    console.error("❌ Please provide filename and message");
    console.log("Usage: step12.ts <filename> <message>");
    process.exit(1);
}

const filename = args[0];
const message = args.slice(1).join(' ');

try {
    writeFileSync(filename, message + '\n');
    console.log(`✅ Wrote to ${filename}: ${message}`);
} catch (error) {
    console.error(`❌ Error writing file: ${error.message}`);
    process.exit(1);
}
```

Try it:
```bash
./step12.ts output.txt "Hello from CLI!"
./step11.ts output.txt
```

## Step 13: Combining Everything

Let's build a simple note-taking CLI that uses everything we've learned:

```typescript
#!/usr/bin/env bun
// notes.ts

import { readFileSync, writeFileSync, existsSync, appendFileSync } from 'fs';

const args = process.argv.slice(2);
const notesFile = 'notes.txt';

const showHelp = () => {
    console.log("📝 Notes CLI");
    console.log("Usage:");
    console.log("  notes.ts add <message>  Add a note");
    console.log("  notes.ts list          Show all notes");
    console.log("  notes.ts clear         Clear all notes");
    console.log("  notes.ts help          Show this help");
};

// Handle help and empty commands
if (args.length === 0 || args.includes('help') || args.includes('--help')) {
    showHelp();
    process.exit(0);
}

const command = args[0];

try {
    if (command === 'add') {
        // Need a message to add
        if (args.length < 2) {
            console.error("❌ Please provide a message to add");
            console.log("Usage: notes.ts add <message>");
            process.exit(1);
        }
        
        const message = args.slice(1).join(' ');
        const timestamp = new Date().toLocaleString();
        const note = `[${timestamp}] ${message}\n`;
        
        // Add to file
        appendFileSync(notesFile, note);
        console.log(`✅ Added note: ${message}`);
        
    } else if (command === 'list') {
        // Check if notes file exists
        if (!existsSync(notesFile)) {
            console.log("📝 No notes yet. Add some with: notes.ts add <message>");
            process.exit(0);
        }
        
        const content = readFileSync(notesFile, 'utf-8');
        console.log("📝 Your notes:");
        console.log(content);
        
    } else if (command === 'clear') {
        // Clear all notes
        writeFileSync(notesFile, '');
        console.log("✅ All notes cleared");
        
    } else {
        console.error(`❌ Unknown command: ${command}`);
        showHelp();
        process.exit(1);
    }
    
} catch (error) {
    console.error(`❌ Error: ${error.message}`);
    process.exit(1);
}
```

Try your notes CLI:
```bash
./notes.ts add "Remember to buy milk"
./notes.ts add "Call mom tomorrow"
./notes.ts list
./notes.ts clear
```

## 🎯 Challenge: File Stats CLI

Now build your own CLI that analyzes text files! Use everything you've learned.

### Requirements:
1. **Accept a filename as argument**
2. **Check if the file exists** 
3. **Count and display:**
   - Number of lines
   - Number of words  
   - Number of characters

### Getting Started:
Create a file called `file-stats.ts` that:
- Takes a filename as the first argument
- Shows help when no arguments or `--help` is provided
- Validates the file exists
- Reads the file and counts lines, words, and characters
- Displays the results nicely

### Example Usage:
```bash
./file-stats.ts document.txt
# Should output something like:
# 📄 File: document.txt
# 📊 Lines: 25
# 📊 Words: 150
# 📊 Characters: 800
```

### Hints:
- Use `content.split('\n').length` for lines
- Use `content.split(' ').length` for a basic word count
- Use `content.length` for characters
- Remember to handle errors and empty files

### Success Test:
Create a test file and verify your CLI works:
```bash
echo -e "Hello world\nThis is a test\nLine three" > test.txt
./file-stats.ts test.txt
```

## What You've Learned

Congratulations! You now understand:
- ✅ How to read command-line arguments with `process.argv`
- ✅ Different argument patterns (`--flags`, values)
- ✅ Input validation and error handling
- ✅ Reading and writing files
- ✅ Building a complete CLI from scratch

## Next Steps

In Phase 2, we'll learn:
- Professional argument parsing libraries
- Better project structure
- More advanced CLI patterns

You're ready to tackle more complex CLI challenges!