---
Primary_category: "[[FILE MANIPULATION]]"
title: MDB
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
  - MicrosoftAccessMDB
cssclasses:
---

###### PRIMARY CATEGORY → [[FILE MANIPULATION]]

***MDB → File extension used by Microsoft Access in versions prior to 2007***

#### .MDB File Manipulation

##### Mdb-Tools

To interact with a *.MDB* file in Linux →

```bash
apt install -y -- mdbtools
```

###### *List MDB Shema*

```bash
mdb-schema <MDB_FILE>
```

###### *List MDB Tables*

```bash
mdb-tables -1 <MDB_FILE>
```

###### *Get MDB Table Content*

```bash
mdb-export <MDB_FILE> <TABLE_NAME>
```

---

#### MDB to SQL

##### Mdb-Tools

###### *Schema Structure*

```bash
mdb-schema <MDB_FILE> mysql > <SQL_FILE>
```

###### *Values to Insert*

```bash
while IFS= read -r _table ; do mdb-export --insert=mysql --quote=\' --bin=octal --date-format=%F backup.mdb "$_table" ; done < <(mdb-tables -1 backup.mdb )
```

> [!DANGER]- *Script*
>
> ```bash
> #!/usr/bin/env bash
>
> exportToSQL ()
> {
>     local mdb=$1 output=${2:--} dialect=${3:-mysql}
>
>     mdb-schema "$mdb" "$dialect"
>
>     while IFS= read -r _table
>     do
>         mdb-export -I "$dialect" -q\' -boctal -D%F "$mdb" "$_table"
>
>     done < <( mdb-tables -1 "$mdb" )
> }
>
> [[ $output != - ]] && exec >"$output"
>
> exportToSQL "$@" || exit 99
> ```
>
> ```bash
> $ bash MDBConv.bash MDB_FILE OUTPUT_FILE BACKEND(mysql, sql...)
> ```
>

---

#### MDB to Json

##### Mdb-Tools

```bash title="mdb-json"
{ while IFS= read -r _table; do mdb-json backup.mdb "$_table"; done < <( mdb-tables -1 backup.mdb ) ; } > backup.json
```

###### Extract Data from JSON

- ***Username and Password → USERNAME:PASSWORD***

> ***e.g. john:password123***

```bash
{ jq --raw-output '"\(.username):\(.password)" | select( . != "null:null")' < <(< backup.json) | cat --language json - } > credentials.txt
```

```bash
{ jq --raw-output '"\(.name):\(.PASSWORD)" | select( . != "null:null")' < <(< backup.json) | cat --language json - } >> credentials.txt
```

- ***User's Name and Lastname***

> ***e.g. JCarter, MSmith***

```bash
{ jq --raw-output '"\(.name | split("")[0])\(.lastname)"' < <(< backup.json) 2> /dev/null } > users.txt
JCarter
```

> ***e.g. john.carter, mark.smith***

```bash
{ jq --raw-output '"\((.name | ascii_downcase)).\((.lastname | ascii_downcase))" | select( . != "null.null")' < <(< backup.json) 2> /dev/null } > users.txt
```
