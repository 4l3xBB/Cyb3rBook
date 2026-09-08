---
Primary_category: "[[PENTESTING]]"
title: "DOCUMENTATION & REPORTING"
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[PENTESTING]]

#### *Notetaking Sample Structure*

> ***Pentest Assessment***

> ***How to properly take notes following a specific structure and organization***

![[DOCUMENTATION & REPORTING-20260505193851815.webp|350]]

> ***Zoom in***

---

#### *Assessment Data Storage Sample Structure*

> ***Pentest Assessment***

> ***How to properly organize all data generated during an assessment***

![[DOCUMENTATION & REPORTING-20260506095534923.webp|300]]

> ***Zoom in***

> [!IMPORTANT]- *Sections*
>
> ###### *Admin*
>
> Scope of Work (SoW) that you're working off of, your notes from the project kickoff meeting, status reports, vulnerability notifications and so on
>
> ###### *Deliverables*
>
> Folder for keeping your deliverables as you work through them. This will often be your report but can include other items such as supplemental spreadsheets and slide decks, depending on the specific client requirements
>
> ###### *Evidence*
>
> - ***Findings***
>
> We suggest creating a folder for each finding you plan to include in the report to keep your evidence for each finding in a container to make piecing the walkthrough together easier when you write the report
>
> - ***Scans***
>
> ***Vulnerability Scans →*** Export files from your vulnerability scanner (if applicable for the assessment type) for archiving
>
> ***Service Enumeration →*** Export files from tools you use to enumerate services in the target environment like Nmap, Masscan, Rumble and so on
>
> ***Web →*** Export files for tools such as ZAP or Burp state files, EyeWitness, Aquatone...
>
> ***AD Enumeration →*** JSON files from BloodHound, CSV files generated from PowerView or ADRecon, Ping Castle data, Snaffler log files, CrackMapExec logs, data from Impacket tools, etc
>
> - ***Notes***
>
> A folder to keep your notes in
>
> - ***OSINT***
>
> Any OSINT output from tools like Intelx and Maltego that doesn't fit well in your notes document
>
> - ***Wireless***
>
> Optional if wireless testing is in scope, you can use this folder for output from wireless testing tools
>
> - ***Logging Output***
>
> Logging output from ***[[#Tmux Logging|TMUX]]***, Metasploit, and any other log output that does not fit the **`Scan`** subdirectories listed above
>
> ***Misc Files***
>
> Web shells, payloads, custom scripts, and any other files generated during the assessment that are relevant to the project
>
> ###### *Retest*
>
> This is an optional folder if you need to return after the original assessment and retest the previously discovered findings. You may want to replicate the folder structure you used during the initial assessment in this directory to keep your retest evidence separate from your original evidence
>

##### *Setup*

```bash
mkdir -p <ENTERPRISE>/{Admin,Deliverables,Evidence/{Findings,Scans/{Vuln,Service,Web,'AD Enumeration'},Notes,OSINT,Wireless,'Logging output','Misc Files'},Retest}
```

> [!DANGER]- *Folder Tree*
>
> ```bash
> <ENTERPRISE>/
> ├── Admin
> ├── Deliverables
> ├── Evidence
> │   ├── Findings
> │   ├── Logging output
> │   ├── Misc Files
> │   ├── Notes
> │   ├── OSINT
> │   ├── Scans
> │   │   ├── AD Enumeration
> │   │   ├── Service
> │   │   ├── Vuln
> │   │   └── Web
> │   └── Wireless
> └── Retest
> ```
>

Then, we can open the **`<ENTERPRISE>`** folder as a vault from *Obsidian*, so we can interact with the notes and folders directly from the command line or inside the *Obsidian* tool

---

#### *Logging*

It becomes essential to save to a log file all scanning and attack attempts we perform during our assessment, including each tool's raw output

Doing so, we have a fallback just in case we missed something during our notetaking

##### *Tmux Logging*

> ***Check out [[TMUX|Tmux]] for more information***

> ***[Tmux Logging](https://github.com/tmux-plugins/tmux-logging)***

![[tmux_log_enable.gif|300]]

> ***Zoom in***

###### *Key Binding*

> ***C ↔ Control***
> ***M ↔ Alt***
> ***S ↔ Shift***

| ***Action*** | ***Shortcut*** |
| --- | --- |
| ***Start/Stop Current Session/Pane Logging*** | **`<prefix> + S-p`** |
| ***Save Visible Pane Content ( Screen Capture )*** | **`<prefix> + M-p`** |
| ***Save Complete Current Pane History ( Retroactive Dump )*** | **`<prefix> + M-S-p`** |
| ***Clear Current Pane History*** | **`<prefix> + M-c`**

###### *Setup*

- ***Cloning the Github Repository***

```bash
git clone https://github.com/tmux-plugins/tmux-logging Tmux-Logging
```

- ***Adding the snippet below to the `~/.tmux.conf` file***

> [!BUG]- *Snippet*
>
> ```bash
> ...<SNIP>...
> # List of plugins
> 
> set -g @plugin 'tmux-plugins/tpm'
> set -g @plugin 'tmux-plugins/tmux-sensible'
> set -g @plugin 'tmux-plugins/tmux-logging'
> 
> # Initialize TMUX plugin manager (keep at bottom)
> run '~/.tmux/plugins/tpm/tpm'
> ```
>

- ***Applying TMUX configuration***

```bash
tmux source ~/.tmux.conf
```

- ***Creating and accessing a new TMUX Session***

```bash
tmux new -s '<TMUX_SESSION_NAME>'
```

- ***Installing previous plugins***

> ***Within the TMUX Session***

```bash
<PREFIX> + Shift + i
```

---

#### *Resources*

***[Black Hill Infosec: How to not suck at reporting](https://www.blackhillsinfosec.com/how-to-not-suck-at-reporting-or-how-to-write-great-pentesting-reports/)***