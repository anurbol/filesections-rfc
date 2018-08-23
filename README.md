# File Sections v 1.0 - RFC

This document suggests a standard/convention of writing data (hereinafter: the Data) to a file.

<!-- todo TOC (when QurDocs ready) -->

## Keywords

Keywords MUST, MUST NOT, SHOULD, SHOULD NOT, MAY have the meaning described [here](https://www.ietf.org/rfc/rfc2119.txt).

## Target Audience

Software developers writing in any programming languages.

## Application Areas

Writing to and reading from files.

## Scope of this Document

The document only describes possible syntax and use cases. Following subjects are not covered:

* implementing a parser (but links to existing ones are [provided](#implementations)).

## Possible Benefits

* easier writing of the Data;
* easy extracting of the Data without using regular expressions;
* easy removing of the Data (e.g. when uninstalling software from an OS, when uninstalling dependencies from a package etc.);
* possibility of as easy navigating in a file contents as in directories;
* easier analyzing of the shared files (multiple parties can write to them), i.e. it is easy to know which parties have already written their data to the file.
  
## Problem

```shell
# .git/hooks/pre-commit (or any other shared file, e.g. .bashrc, .profile, etc.)

# the code added by library A
some_command_from_library_A

# the code added by library B
some_command_from_library_B
```

In the code above we see that there are two pieces added by different parties: libraries A and B.

While it is trivial to write to the file, it is not so when editing or removing previously written data is required. Usually developers go with wrapping the Data with identifying blocks and using regex solution:

```shell
# .git/hooks/pre-commit

# [A lib's code starts]
some_command_from_library_A
# [A lib's code ends]

#B_start
some_command_from_library_B
#B_end
```

The libraries A and B can, with the help of regular expressions, find their data, and remove it. You can see, that there is no consistency in style. However, is it only matter of style/aesthetics? Doesn't seem so. The above example makes it hard to do the following:

* Removing the Data. Other than having to use regular expressions, there is a theoretical possibility of removing excessive parts of the file if one of two things happens: either the developer used non-unique wrapping identifiers, or his regex was not made correctly. The developer has to spend his time to think about these cases to prevent them, which means, not only more precious time is spent, but also, the human factor is still there.
* Get all parties, who inserted their code, find out if a competing/collaborating library's code already presented in the file.
* Sort code blocks of different parties (sometimes, in rare cases, may appear to be a useful feature).
* Probably most important, navigating the file. It is absolutely possible to navigate directories. This is probably one of the earliest feature available in programming history. There are plenty of ways to do that, in any language. However, files can still become very big, containing many discrete concerns. Yet there is no way to get a structured representation of a file. Files and directories are only concepts, they both contain bits of data, and having the last and the main data container (a file) unstructured seems not right.

## Solution

There should be a consistent style for placing a data from different parties/vendors to a shared file.

```shell
# [filesection vendor=A]
some_command_from_library_A
# [/filesection]

# [filesection vendor=B]
some_command_from_library_B
# [/filesection]
```

The above code allows to implement a tool for enabling features listed previously.

### More Examples

#### Nested File Sections

```shell
# [filesection vendor=A]
some_command_from_library_A
# [/filesection]

# [filesection vendor=B]
some_command_from_library_B

++    # [filesection name=other_command]
++    other_command_from_library_B
++    # [/filesection]

# [/filesection]

```

#### Multiple File Sections of one vendor

```shell
# [filesection vendor=A]
some_command_from_library_A
# [/filesection]

# [filesection vendor=B]
some_command_from_library_B
# [/filesection]

++ # [filesection vendor=A name=other_command]
++ other_command_from_library_A
++ # [/filesection]

```

## Specification

1. **NAMING**

    1.1. `[filesection]` and `[/filesection]` constructs are called _**file section tag**_ (an opening and a closing one, respectively). Everything in between an opening and a closing File Section Tags is called a _**"file section"**_. Word "filesection" MAY be used instead of "file section" as well. Word "section" MAY be used instead of "file section" in an appropriate context.

2. **SYNTAX**

    2.1. **TAGS**

    2.1.1. The starting and closing tags MUST be named "filesection":

    ```shell
    # [filesection vendor=Library_X]
    ...
    # [/filesection]
    ```

    2.1.2. The closing tag MUST always be present and MUST have exactly one slash before its name.

    2.1.3. The tags MUST be written as inline comments.

    _Example for a bash script:_

    ```bash

    # [filesection name=bash_example]
    echo "i am written in bash"
    # [/filesection]
    ```

    _Example for a javascript file:_
    ```js
    // [filesection name=javascript_example]
    console.log("i am written in javascript")
    // [/filesection]
    ```

    2.2. **ATTRIBUTES**

    2.1.1. Sections MUST have at least one attribute "vendor" or "name".

    <!-- markdownlint-disable MD033 -->
    <details>
    <summary>Reason</summary>

    The "vendor" and "name" attributes' values are *file section identifiers*. They serve to the very purpose of the "file section" concept: identifying and structuring an unstructured file content.

    The <b>"vendor"</b> attribute may help to recognize the author of the content in a shared file.

    The <b>"name"</b> attribute can help to identify the purpose and the matter of the content. It can be used in cases including, but not limited to:
    <ul>
    <li>in non-shared files (one vendor writing to the file)</li>
    <li>inside a nested sections.</li>
    </ul>

    <br>

    </details>
    <!-- markdownlint-enable MD033 -->

    2.1.2. Sections MAY have arbitrary attributes, other than "vendor" and "name".

    2.3. **ATTRIBUTE VALUES**

    2.3.1. Strings containing spaces MUST be wrapped by single or double quotes:
    ```shell
    # [filesection name=abc info="quotes are allowed"]
    ...
    # [/filesection]
    ```
    2.3.2. Values of "vendor" and "name" attributes SHOULD always be written without spaces (in camelCase or snake_case)

    <!-- markdownlint-disable MD033 -->
    <details>

    <summary>
    Reason
    </summary>

    The "vendor" and "name" attributes SHOULD contain a machine-friendly identifier.

    </details>

    <!-- markdownlint-enable MD033 -->

    2.3.3. Dash-case, aka kebab-case MUST NOT be used and supported at least if without quotes.

    _good_:

    ```shell
    # [filesection name=abc_def]
    ...
    # [/filesection]
    ```
    _bad_:

    ```shell
    # [filesection name=abc-def]
    ...
    # [/filesection]
    ```
    _ok_:

    ```shell
    # [filesection name="abc-def"]
    ...
    # [/filesection]
    ```

    <!-- markdownlint-disable MD033 -->
    <details>
    <summary>
    Reason
    </summary>
    To promote common style.
    </details>
    <!-- markdownlint-enable MD033 -->

    2.2. Nesting

    2.1.1. Sections MAY be nested in each other:

    ```shell
    # [filesection vendor=abc]
        # [filesection name=def]
        echo "i am written in bash"
        # [/filesection]
    # [/filesection]
    ```

## Implementations

Javascript: TODO

## Status of this Document

This document follows the [Semantic Versioning](https://semver.org/) model. Please look at the version (in the main title) of this document to find out status information.
