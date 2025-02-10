# Macro Expander
The Macro Expander project is a C-based program designed to process and expand macros in text files. It reads input strings, processes them for user-defined macros, handles escaped characters, and removes comments. The project provides functionality to handle macro definitions, macro replacements, and processes complex use cases such as handling escaped characters in macros and removing comments effectively.

## Project Overview
In many programming and text-processing environments, macros are used to define reusable expressions or functions that can be expanded in place. This project implements a macro expander that simulates such behavior for text input, supporting the following features:

- Macro Definitions: You can define macros using the \def syntax, followed by the macro name and the macro value in curly braces {}. Macros can be expanded multiple times across the input text.

- Macro Expansions: Once defined, macros are automatically expanded when encountered in the input text. The program replaces occurrences of the macro names with their corresponding values.

- Handling Escaped Characters: The expander can correctly process escaped characters like \% or \\ to ensure that they are not treated as special symbols, such as the comment delimiter %.

- Comment Removal: The program can process comments, which start with a % symbol and continue until the end of the line. It handles situations where % characters are escaped, ensuring they are not mistakenly treated as comment markers.

- Input Processing: The program can read input from files or from standard input. It handles input redirection (<), so users can easily process files without manually providing input each time.

## Key Features:
- Macro Definition: \def{MACRO}{VALUE}
- Comment Handling: Skips over comments that start with %, but only if the % is not escaped.
- Whitespace Handling: When processing comments, the program also ignores spaces, tabs, and newlines until it finds the next non-blank character.
- Escaped Characters: Handles escaped characters like \%, \\, and \n within macro values and definitions.

## Use Cases
- Text Replacement: This tool can be used to define reusable text fragments and replace them within a document. For example, if you define a macro for a certain piece of text, you can use it repeatedly throughout your file without manually retyping it each time.

- Preprocessing: Much like preprocessors in C/C++, this tool can be used to process files where macro definitions are expanded into final content before further processing or output.

- Dynamic Content Generation: It can also be used in situations where certain patterns or replacements need to be dynamically applied to input data, such as templating systems or configuration files.

- Customizable Macros: The flexibility of defining and expanding macros makes it useful for customizable templating systems or even as part of a larger system for data preprocessing.

## Summary of Functionality:
- The program starts by reading input, either from files or standard input.
- It then processes the input string, handling macro definitions, expansions, and removing comments.
- The macro system supports the use of special characters, and the program ensures that escaped characters are correctly processed.
- The final output is the text with macros expanded and comments removed, suitable for further processing or as the final output.
