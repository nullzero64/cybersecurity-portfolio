# File Permissions in Linux

## Project Description
Managed and changed file permissions for User, Group, and Other in Linux.

---

## Check File and Directory Details

To list files and directories with their permissions, use:

```bash
ls -la
```

Here’s an example output:
<img width="774" height="315" alt="image" src="https://github.com/user-attachments/assets/f2144e21-ee68-4cdf-9cc6-861403579997" />

---

## Describe the Permissions String

The first 10 characters in `ls -l` output show file type and permissions:
- `d` = directory,
- `r` = read, `w` = write, `x` = execute, `-` = no permission

Each set of three indicates User, Group, and Other permissions.

---

## Change File Permissions

To remove write permission from Other for `project_k.txt` and remove read permission from Group for `project_m.txt`:

```bash
chmod o-w project_k.txt
chmod g-r project_m.txt
ls -la
```

Here are the results:
<img width="975" height="806" alt="image" src="https://github.com/user-attachments/assets/90e2e0fd-a5d1-4185-ab78-eb1181137860" />

---

## Change File Permissions on a Hidden File

To remove write permissions from both User and Group, and then grant Group read permissions for `.project_x.txt`:

```bash
chmod u-w .project_x.txt
chmod g-w .project_x.txt
chmod g+r .project_x.txt
ls -la
```

Results:
<img width="975" height="802" alt="image" src="https://github.com/user-attachments/assets/f64f9b05-f278-4acd-b08b-a797abc9c815" />

---

## Change Directory Permissions

To remove execution permission from Group for the `drafts` directory:

```bash
chmod g-x drafts/
ls -la
```

Results:
<img width="975" height="567" alt="image" src="https://github.com/user-attachments/assets/b8f601d3-2710-46dd-94b0-8b6d582bcde0" />

---

## Summary

This activity demonstrates how to change permissions for different types of users in Linux, including files, directories, and hidden files. You learned to use `chmod` for various permission modifications and verified changes using `ls -la`.
