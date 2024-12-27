# Here-string & Here-doc in Linux

## Introduction

```sh
ssh -T dinhphu28@example.com 'touch abc1.txt'
ssh -T dinhphu28@example.com 'touch abc2.txt'
```

Instead of above commands, we can use only 1 command with ***heredoc***:

```sh
ssh -T dinhphu28@example.com << EOF
touch abc1.txt
touch abc2.txt
EOF
```

## Here doc

### Syntax

> A heredoc consists of the '<<' redirection operator, followed by a delimiter token.
>
> The delimiter token can be any value as long as it is unique enough that it won't appear within the content.

E.g:

```sh
cat << EOF
This is
super user
from
dinhphu28
EOF
```

### Tab Suppression

To suppress the tab indentations in the output, prefix the redirection operator '<<' with a dash symbol

```sh
cat <<- EOF
This is
		super user
	from
	dinhphu28
EOF
```

### Paramerter substitution

```sh
cat << EOF
This is $USER
aka lm38 superuser
EOF
```

### Command substitution

```sh
cat << EOF
This is list of files $(ls)
EOF
```

### Passing arguments to function

Example, considering this function:

```sh
read_name_and_age() {
	read -p "Name: " z_name
	read -p "Age: " z_age
	echo "Hello $z_name, your age is $z_age"
}
```

Invoking:

```sh
read_name_and_age << EOF
Anthony
25
EOF
```

### Disable Block of Code using heredoc (comment out)

Instead of using '#' to comment out the code, we can use heredoc like this with dummy command `:`

```sh
#!/bin/bash
# This is a comment
# This is a comment too

: << 'DISABLED'
This is a comment
This is a comment too
DISABLED

echo "Hello, world!"
```

Result is only: `Hello, world!`

## Here string

> Here string is similar to heredoc, but it's much simpler.
>
> Here string does not need a delimiter token.

### Syntax

```sh
COMMAND <<< $VAR
```

```sh
base64 <<< "Hello, world!"
```

