# ZSH

### Edit current command with text editor (e.g. neovim)

##### Editing

Use `Ctrl + X` then `Ctrl + E`

##### Set editor to neovim

```sh
export EDITOR="nvim"
export VISUAL="nvim"
```

You can add below lines to `~/.zshrc`

### Using packages installed by BREW with root priviliges (sudo)

Open file `/etc/sudoers`

If not found this file, can use command (I remember that):

```sh
sudo vim sudoers
```

Then find this line

```conf
Defaults secure_path="<something here>"
```

Then add `:/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin/`

Result look like this:

```conf
Defaults secure_path="/usr/sbin:/usr/bin:/sbin:/bin:/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin/"
```

### Read from user input (e.g. password, ...)

```sh
read -r "username?Enter username: " && \
read -rs "password?Enter password: "
```

Result:

```sh
echo $username
echo $password
```

> Remember to `unset password` for security

---

### CURL with parallel

#### Use `xargs`

```sh
#!/bin/bash
ids=("340935" "340134")

fetch_resource() {
	id=$1
	curl -X 'GET' "http://example.com/resources/$id" \
		-H 'X-API-KEY: api-key-1234' | jq
}

export -f fetch_template

printf "%s\n" "${ids[@]}" | xargs -P 12 -I {} bash -c 'fetch_resource "$@"' _ {}

```

#### Use jobs

```sh
#!/bin/bash
ids=("340935" "340134")

# Function to make the curl request
fetch_resource() {
	id=$1
	response=$(curl -X GET "http://example.com/resources/$id" \
		-H 'X-API-KEY: api-key-1234')
	echo "$response" | jq | tee "output/resource_$id.json" | jq
}

# Limit for parallel jobs
parallel_limit=12
job_count=0

for id in "${ids[@]}"; do
	fetch_resource "$id" &
	job_count=$((job_count + 1))

	if ((job_count >= parallel_limit)); then
		wait -n
		job_count=$((job_count - 1))
	fi
done

# Wait for all remaining jobs to finish
wait

```

