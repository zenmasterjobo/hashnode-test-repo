---
title: "Leveraging Linux Internals to Supercharge Osquery Malware Detection"
datePublished: Mon Sep 30 2024 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm2ayfymq000209l37rjqdnes
slug: leveraging-linux-internals-to-supercharge-osquery-malware-detection
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1729027687202/9c0a912f-daaa-4847-a1d9-a517f9da2f61.webp

---


## Introduction
This post outlines what I believe to be a novel way to overcome the limitations of the osquery yara scanning table to find fileless malware on Linux operating systems.

## Background
### What is osquery?
osquery is a powerful open source toolset that exposes operating systems in a way that allows them to be queried with SQL. There are myriad use cases of this instrumentation, but we primarily use it to ask security relevant questions of our hosts. Over the last 10 years, many developers have enhanced the capabilities of the tool and many companies have grown around providing advanced services and features for the platform. For the purposes of this post, we are mainly concerned about the osquery YARA scanning module.

### What is YARA?
YARA is a pattern-matching language primarily used for malware identification and detection. YARA rules allow InfoSec researchers to classify malware based on certain characteristics or patterns such as strings or byte sequences. The flexibility and performance of the YARA language enables very fast and accurate scanning with relatively low performance impact compared to traditional AV or operating system (OS) based pattern matching like grep.

### What is '/proc'?
In Linux, the /proc filesystem is a virtual/abstract file system that provides an interface to kernel data structures and information about the OS. It doesn't represent actual files on disk but rather dynamically generates information about the kernel, processes, and various system parameters. Essentially, /proc is an example of how “everything on Linux is a file".

## How is YARA used in osquery
osquery supports a YARA table. This table takes in a mandatory file path argument, as well as a rules definition. The documentation can be seen here. The out-of-the-box behavior of this table means that there must be a file on disk for you to scan with the YARA rule, and this feature works very well.

The required file path argument is also quite a limitation to detection engineers because many threat actors employ fileless or in-memory malware to hide their implants and ultimately, avoid detection. Thinking about this problem, I came up with a novel way to leverage the virtual Linux filesystem 'proc' to satisfy the file path requirement while also scanning for fileless implants.

## The Problem
There are a variety of ways to hide a process on the Linux OS. For the purposes of this article we will wave our hands and assume our attacker has either dropped a file, executed it and unlinked the binary from disk, or has utilized the dynamic loader via dlopen() to shove the bytes into memory prior to execution. Either way, there is now an unknown malicious process running on our machine, and the file backing it is no longer on disk.

Using osquery itself, we can usually find these processes:

```sql
SELECT * FROM processes WHERE on_disk=1;
```

This very basic query will return all processes where the original binary is no longer present on the machine. For more information, there are many blogs that go into great detail about finding orphaned processes using osquery and other tables and heuristics.

Knowing that there is an unlinked process running on our machine is great, but how can we prove that this orphaned process is actually malicious? Typically, we could run another osquery against this process using the YARA table. Again, we will skip the deep technical details about choosing the right YARA rule, and provide a basic example.

```sql
SELECT * FROM yara where file_path="..." AND sigrule IN ('
rule bad_stuff:
{
    meta:
        description = "This is just an example"
        threat_level = “midnight”

    strings:
        $a = {41 41 41 ?? 41 41 41 41}
        $b = "I am a malicious file"

    condition:
        $a or $b
}')
AND matches = ‘rule_bad_stuff’;
```
The problem for our detection engineer should now be apparent, **how can we provide the mandatory `file_path` argument to the yara query if there is no file on disk?**