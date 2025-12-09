## Day 2 - Create temporary user 'siva'

### As part of the temporary assignment to the Nautilus project, a developer named siva requires access for a limited duration. To ensure smooth access management, a temporary user account with an expiry date is needed. Here's what you need to do: Create a user named siva on App Server 3 in Stratos Datacenter. Set the expiry date to 2024-02-17, ensuring the user is created in lowercase as per standard protocol. Note: You can find the infrastructure details by clicking on the Details of all Users and Servers button on the top-right section of the page.

## 1) Get the server address & connect
```
ssh adminuser@app-server-3.stratos.example
```
### or by IP
```
ssh adminuser@10.0.3.5
```
### You need a user with sudo privileges (or be root) to create accounts.

## 2) Check if siva already exists
```
getent passwd siva
```
### If it returns line below then the user exists.
`siva:x:1002:1002:...`

### If no output, the user does not exist and you're clear to create.

## 3) Create the user siva (lowercase) with expiry 2024-02-17 (exactly as requested)
```
sudo useradd -m -s /bin/bash -e 2024-02-17 siva
```
### Explanation:

### `sudo` — run as privileged user.

### `useradd` — tool to add users.

### `-m` — create the user’s home directory /home/siva.

### `-s` /bin/bash — set login shell to bash.

### `-e` 2024-02-17 — account expiry date in YYYY-MM-DD. Because this date is in the past (Feb 17, 2024), the account will be expired immediately and the user cannot log in until the expiry is set to a future date or removed.

## 4) Set an initial password and force password change at first login

### # set password interactively
```
sudo passwd siva
```
### then force user to change on first login:
```
sudo chage -d 0 siva
```

## 5) Verify the user and expiry

## Check account existence and expiry info:

```
getent passwd siva
```

### show account expiry and password info
```
sudo chage -l siva
```

