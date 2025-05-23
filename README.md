# Beginner's Guide to Writing a pyPreservica Script (with ChatGPT & Documentation)

## Prerequisites
- Python 3.6 or later (Python is a beginner-friendly programming language that uses simple, readable words and doesn't require complex punctuation)
- pip (comes with Python; it's a tool that helps you install Python libraries)
- A Preservica account (you'll need your tenant name, username, password, and server URL)
- A text editor:
  - Windows: Notepad++ or VS Code (recommended)
  - Linux/Ubuntu: VS Code, nano, or gedit

## Step 1: Set Up Your Project Environment

### What is a Virtual Environment?
A virtual environment is like a separate, clean kitchen for each of your cooking projects. It keeps your ingredients (Python libraries) separate so they don't mix with other projects. This is important because:

- Different projects might need different versions of the same library
- It keeps your main Python installation clean
- It makes it easier to share your project with others

### Create a Virtual Environment

#### On Windows:

```bash
# Create a new directory for your project
mkdir preservica-scripts
cd preservica-scripts

# Create a virtual environment
python -m venv venv

# Activate the virtual environment
venv\Scripts\activate
```

#### On Linux/Ubuntu:

```bash
# Create a new directory for your project
mkdir preservica-scripts
cd preservica-scripts

# Install python3-venv if not already installed
sudo apt update
sudo apt install python3-venv

# Create a virtual environment
python3 -m venv venv

# Activate the virtual environment
source venv/bin/activate
```

### What is a Library?
A library is like a pre-written set of tools that someone else has created for you to use. Instead of writing everything from scratch, you can use these ready-made tools. For example:
- `pyPreservica` is a library that helps you talk to Preservica
- `python-dotenv` is a library that helps you manage secret information (like passwords)

Think of libraries like kitchen appliances: you could make bread by hand, but using a bread maker (a "library") makes it much easier!

### Install Required Libraries
```bash
# Make sure your virtual environment is activated (you should see (venv) at the start of your command line)
#### On Windows

```bash
pip install pyPreservica python-dotenv

```

#### On Linux/Ubuntu

```bash
pip3 install pyPreservica python-dotenv
```

## Step 2: Create Your .env File

### What is a .env File?
A `.env` file is like a secret notebook where you keep important information (like your Preservica username and password). It's called a "dot env" file because:
- It starts with a dot (.)
- "env" stands for "environment"
- It keeps your secrets separate from your code

This is important because:
- You don't want to accidentally share your password in your code
- It makes it easier to use different passwords for different environments (like testing vs. production)
- It's a security best practice

### Creating .env File

#### On Windows:
1. Open Notepad++ or VS Code
2. Create a new file
3. Add your credentials:
   ```plaintext
   USERNAME=your_username
   PASSWORD=your_password
   TENANT=your_tenant
   SERVER=your_server_url
   ```
4. Click "Save As"
5. Set "Save as type" to "All Files (*.*)"
6. Name the file exactly `.env` (including the dot)
7. Save it in your project directory

#### On Linux/Ubuntu:
1. Open a text editor (choose one):
   ```bash
   # Using nano:
   nano .env
   
   # Or using gedit:
   gedit .env
   
   # Or using VS Code:
   code .env
   ```
2. Add your credentials:
   ```plaintext
   USERNAME=your_username
   PASSWORD=your_password
   TENANT=your_tenant
   SERVER=your_server_url
   ```
3. Save the file:
   - In nano: Press Ctrl+X, then Y, then Enter
   - In gedit: Click Save
   - In VS Code: Press Ctrl+S

4. Set proper file permissions (Ubuntu security requirement):
   ```bash
   chmod 600 .env
   ```

## Step 3: Write Your First Script

### What is a Script?
A script is just a text file containing Python instructions. It's like a recipe that your computer follows step by step. When you run a script, your computer reads these instructions and performs the actions you've specified.

### What is an API?
An API (Application Programming Interface) is like a waiter in a restaurant:
- You (the customer) don't go into the kitchen (Preservica's system)
- Instead, you tell the waiter (the API) what you want
- The waiter goes to the kitchen and brings back what you asked for

In our case, `pyPreservica` is like a special waiter that knows how to talk to Preservica's system.

### Create Your Script

#### On Windows:
1. Open Notepad++ or VS Code
2. Create a new file
3. Save it as `my_script.py` in your project directory
4. Add the code (shown below)

#### On Linux/Ubuntu:
```bash
# Using nano:
nano my_script.py

# Or using gedit:
gedit my_script.py

# Or using VS Code:
code my_script.py
```

Add this code to your script:
```python
import os
from dotenv import load_dotenv
from pyPreservica import *

# Load your secret info from the .env file
load_dotenv(override=True)
USERNAME = os.getenv('USERNAME')
PASSWORD = os.getenv('PASSWORD')
TENANT = os.getenv('TENANT')
SERVER = os.getenv('SERVER')

# Check that all required info is present
if not USERNAME or not PASSWORD or not TENANT or not SERVER:
    print("Missing one or more environment variables. Please check your .env file.")
    exit(1)

# Create the EntityAPI client object (recommended by documentation)
client = EntityAPI(username=USERNAME, password=PASSWORD, tenant=TENANT, server=SERVER)

# Use None for the root folder, or specify a folder reference
folder_ref = None  # or e.g. 'bb45f999-7c07-4471-9c30-54b057c500ff'

# List the first 5 assets in the repository, following the official filter usage
count = 0
for asset in filter(only_assets, client.all_descendants(folder_ref)):
    # Fetch the full asset object if you need more details (optional)
    asset_full = client.asset(asset.reference)
    print(f"Asset ID: {asset_full.reference}, Title: {asset_full.title}")
    count += 1
    if count >= 5:
        break
```

## Step 3.5: (OPTIONAL) Adding Command-Line Arguments to Your Script

### What are Command-Line Arguments?
Command-line arguments are like options you can give to your script when you run it. Instead of changing the code every time you want to do something different, you can tell the script what to do when you run it. For example:
- Instead of hardcoding the number of assets to list, you can specify it when running the script
- Instead of always using the root folder, you can specify a different folder
- You can add a flag to show more detailed information

### Enhancing Your Script with argparse

Here's how to modify your script to use command-line arguments:

```python
import os
import argparse
from dotenv import load_dotenv
from pyPreservica import *

# Set up command-line argument parsing
parser = argparse.ArgumentParser(description='List assets from Preservica')
parser.add_argument('--folder', type=str, help='Folder reference to list assets from (default: root folder)')
parser.add_argument('--limit', type=int, default=5, help='Number of assets to list (default: 5)')
parser.add_argument('--verbose', action='store_true', help='Show more detailed information')
args = parser.parse_args()

# Load your secret info from the .env file
load_dotenv(override=True)
USERNAME = os.getenv('USERNAME')
PASSWORD = os.getenv('PASSWORD')
TENANT = os.getenv('TENANT')
SERVER = os.getenv('SERVER')

# Check that all required info is present
if not USERNAME or not PASSWORD or not TENANT or not SERVER:
    print("Missing one or more environment variables. Please check your .env file.")
    exit(1)

# Create the EntityAPI client object
client = EntityAPI(username=USERNAME, password=PASSWORD, tenant=TENANT, server=SERVER)

# Use the folder reference from command line, or None for root folder
folder_ref = args.folder

# List assets in the repository, following the official filter usage
count = 0
for asset in filter(only_assets, client.all_descendants(folder_ref)):
    # Fetch the full asset object if you need more details
    asset_full = client.asset(asset.reference)
    
    if args.verbose:
        # Show more detailed information
        print(f"Asset ID: {asset_full.reference}")
        print(f"Title: {asset_full.title}")
        print(f"Description: {asset_full.description}")
        print(f"Security Tag: {asset_full.security_tag}")
        print("-" * 50)
    else:
        # Show basic information
        print(f"Asset ID: {asset_full.reference}, Title: {asset_full.title}")
    
    count += 1
    if count >= args.limit:
        break
```

### Running Your Script with Arguments

#### On Windows:
```bash
# List 5 assets from root folder (default)
python my_script.py

# List 10 assets from a specific folder
python my_script.py --folder "bb45f999-7c07-4471-9c30-54b057c500ff" --limit 10

# Show detailed information for 3 assets
python my_script.py --limit 3 --verbose

# Get help on available options
python my_script.py --help
```

#### On Linux/Ubuntu:
```bash
# List 5 assets from root folder (default)
python3 my_script.py

# List 10 assets from a specific folder
python3 my_script.py --folder "bb45f999-7c07-4471-9c30-54b057c500ff" --limit 10

# Show detailed information for 3 assets
python3 my_script.py --limit 3 --verbose

# Get help on available options
python3 my_script.py --help
```

### Understanding the Arguments
- `--folder`: Optional folder reference to list assets from
- `--limit`: Number of assets to list (default: 5)
- `--verbose`: Flag to show more detailed information
- `--help`: Shows help message with all available options

### Benefits of Using Command-Line Arguments
1. **Flexibility**: Change script behavior without editing code
2. **Reusability**: Use the same script for different purposes
3. **Documentation**: Built-in help system with `--help`
4. **Automation**: Easier to use in automated scripts or workflows

Note: This section is optional. You can start with the basic script and add command-line arguments later when you're comfortable with the basics.

## Step 4: Running Your Script

### What is Running a Script?
Running a script means telling your computer to follow the instructions in your Python file. It's like giving your recipe to a chef and asking them to cook it.

### Run Your Script

#### On Windows:
1. Make sure your virtual environment is activated (you should see `(venv)` at the start of your command line)
2. Run your script:
   ```bash
   python my_script.py
   ```

#### On Linux/Ubuntu:
1. Make sure your virtual environment is activated (you should see `(venv)` at the start of your command line)
2. Run your script:
   ```bash
   python3 my_script.py
   ```

### Understanding the Output
When you run the script, you should see a list of assets from your Preservica system. Each line will show:
- The Asset ID (a unique identifier)
- The Asset Title (the name of the asset)

If you see an error message, don't worry! This is normal when learning to program. Common issues include:
- Missing or incorrect credentials in your `.env` file
- Virtual environment not activated
- Required libraries not installed

## Step 5: Deactivating the Virtual Environment

### What is Deactivating?
Deactivating is like cleaning up your kitchen after cooking. It tells your computer that you're done working in this project's environment.

When you're done (on both Windows and Linux/Ubuntu):
```bash
deactivate
```

## Appendix: Using ChatGPT with pyPreservica Documentation

### What is ChatGPT?
ChatGPT is an AI assistant that can help you write, debug, and explain code. Think of it as a very knowledgeable programming tutor who can:
- Answer your questions about Python and pyPreservica
- Help you write code
- Explain how code works
- Suggest improvements to your code

Note: While ChatGPT is helpful, always verify its suggestions against the official documentation.

### How to Use ChatGPT Effectively with pyPreservica

1. **Read the Documentation First**
   - Always check the [pyPreservica documentation](https://pypreservica.readthedocs.io/en/latest/entity.html) first
   - This helps you understand what's possible and what to ask for

2. **Ask ChatGPT with Context**
   Instead of asking:
   ```
   "How do I download a file from Preservica?"
   ```
   
   Ask with context:
   ```
   "I'm using pyPreservica and have read the documentation at https://pypreservica.readthedocs.io/en/latest/entity.html#downloading-files. How do I write a script to download a file from Preservica using the download() method?"
   ```

3. **Example ChatGPT Prompt**
   ```
   I'm a beginner using pyPreservica. I've read the documentation at https://pypreservica.readthedocs.io/en/latest/entity.html#downloading-files. Can you help me write a script that:
   1. Connects to Preservica using environment variables (like in the example script)
   2. Downloads a specific asset's file
   3. Includes error handling
   Please explain each part of the code.
   ```

4. **Review ChatGPT's Response**
   - Compare the code with the official documentation
   - Test the code in your virtual environment
   - Ask follow-up questions if needed

### Example: Downloading a File
Here's an example script that ChatGPT might help you create (based on the documentation):

```python
import os
from dotenv import load_dotenv
from pyPreservica import *

# Load environment variables (same as our example script)
load_dotenv(override=True)
USERNAME = os.getenv('USERNAME')
PASSWORD = os.getenv('PASSWORD')
TENANT = os.getenv('TENANT')
SERVER = os.getenv('SERVER')

if not all([USERNAME, PASSWORD, TENANT, SERVER]):
    print("Missing one or more environment variables. Please check your .env file.")
    exit(1)

try:
    # Create the EntityAPI client (same as our example script)
    client = EntityAPI(username=USERNAME, password=PASSWORD, tenant=TENANT, server=SERVER)
    
    # Fetch an asset (replace with your asset reference)
    asset = client.asset("your-asset-reference-here")
    
    # Download the file
    filename = client.download(asset, "downloaded_file.jpg")
    print(f"Successfully downloaded: {filename}")
    
except Exception as e:
    print(f"An error occurred: {str(e)}")
```

## Tips for Success

1. **Always Use a Virtual Environment**
   - Keeps your project dependencies isolated
   - Makes it easier to manage different Python versions and packages

2. **Keep Your .env File Secure**
   - Never commit it to version control
   - Add `.env` to your `.gitignore` file
   - On Linux/Ubuntu, use `chmod 600 .env` to set proper permissions

3. **Use ChatGPT Effectively**
   - Always provide context by linking to the documentation
   - Ask specific questions
   - Test the code ChatGPT provides
   - Ask for explanations of complex parts

4. **Start Simple**
   - Begin with the example script provided
   - Gradually add more complex functionality
   - Test each step before moving on

5. **Documentation is Your Friend**
   - Bookmark the [pyPreservica documentation](https://pypreservica.readthedocs.io/en/latest/entity.html)
   - Refer to it often
   - Use it to verify ChatGPT's suggestions

## Need More Help?

- Check the [pyPreservica documentation](https://pypreservica.readthedocs.io/en/latest/entity.html)
- Ask ChatGPT with specific questions and documentation links
- Start with the example script and build up to more complex ones
- Test your code in a development environment before using it in production

Remember: Always test ChatGPT's suggestions against the official documentation and in a safe environment before using them in production.
