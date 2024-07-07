# Day 1

## YAML & Linting Tutorial

### Introduction to YAML

YAML (YAML Ain't Markup Language) is a human-readable data serialization standard that is commonly used for configuration files and data exchange between languages with different data structures. In this tutorial, we will cover the essentials of YAML, its syntax, and how to use linting tools to ensure your YAML files are properly formatted.

### Table of Contents
1. [Introduction to YAML](#introduction-to-yaml)
2. [Is YAML Case-Sensitive?](#is-yaml-case-sensitive)
3. [Serialization and Deserialization](#serialization-and-deserialization)
4. [Data Structures in YAML](#data-structures-in-yaml)
5. [Quoting in YAML](#quoting-in-yaml)
6. [Key-Value Pairs](#key-value-pairs)
7. [Whitespace and Alignment](#whitespace-and-alignment)
8. [YAML File Format](#yaml-file-format)
9. [Using Linting Tools](#using-linting-tools)
10. [Example YAML Documents](#example-yaml-documents)

### Is YAML Case-Sensitive?

YAML is case-sensitive. This means that `name` and `Name` are considered different keys. Here’s an example:

```yaml
name: Prerit
Name: Rishabh
```

Both `name` and `Name` are valid keys and can exist together in a YAML file.

### Serialization and Deserialization

Serialization is the process of converting a data structure into a format that can be easily stored or transmitted, and deserialization is the reverse process. In YAML, this means converting data structures into a human-readable format and back again.

For example, a Python dictionary can be serialized into YAML and then deserialized back into a dictionary:

#### Python Dictionary
```python
data = {
    'name': 'Prerit',
    'age': 25,
    'languages': ['Python', 'JavaScript']
}
```

#### YAML Representation
```yaml
name: Prerit
age: 25
languages:
  - Python
  - JavaScript
```

### Data Structures in YAML

YAML supports several data structures, including scalars (single values), lists, and dictionaries (key-value pairs).

#### Scalars
A scalar is a single value, such as a string or number.

```yaml
name: Prerit
age: 25
```

#### Lists
A list is an ordered collection of values.

```yaml
languages:
  - Python
  - JavaScript
  - Go
```

#### Dictionaries
A dictionary is a collection of key-value pairs.

```yaml
person:
  name: Prerit
  age: 25
  languages:
    - Python
    - JavaScript
    - Go
```

### Quoting in YAML

Quoting is used in YAML to define strings that include special characters or to preserve spaces.

#### Unquoted Strings
YAML can usually infer the type of data without quotes.

```yaml
name: Prerit
```

#### Single Quotes
Use single quotes for strings that contain special characters or spaces.

```yaml
name: 'Prerit'
bio: 'Engineer & Developer'
```

#### Double Quotes
Double quotes are used for strings that include escape characters.

```yaml
greeting: "Hello, \"World\"!"
```

### Key-Value Pairs

YAML uses key-value pairs to represent data. The key is a unique identifier, and the value can be any data type.

#### Example
```yaml
name: Prerit
age: 25
```

In this example, `name` and `age` are keys, and `Prerit` and `25` are their respective values.

### Whitespace and Alignment

Whitespace is critical in YAML, and alignment (indentation) is used to represent the structure.

#### Indentation
Use spaces (not tabs) to indent your YAML files.

```yaml
person:
  name: Prerit
  age: 25
  languages:
    - Python
    - JavaScript
```

### YAML File Format

YAML files can have either the `.yaml` or `.yml` extension. Both are acceptable and will be recognized by YAML parsers.

### Using Linting Tools

Linting tools help to ensure your YAML files are properly formatted. They check for syntax errors and provide feedback.

#### Example Linting Tool
One popular linting tool is `yamllint`. You can install it using pip:

```bash
pip install yamllint
```

#### Using `yamllint`
Create a YAML file `example.yaml`:

```yaml
name: Prerit
age: 25
languages:
  - Python
  - JavaScript
```

Run `yamllint` on the file:

```bash
yamllint example.yaml
```

### Example YAML Documents

#### Basic Configuration File
```yaml
version: 1.0
service:
  name: ExampleService
  port: 8080
  database:
    host: localhost
    port: 5432
    username: user
    password: pass
```

#### Multi-Document File
YAML allows multiple documents to be included in a single file, separated by `---`.

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: myfrontend
    image: docker.io/nginx
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: mypod
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
