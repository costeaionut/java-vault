---
title: 
draft: true
---

### 1. Intro

- **JVM** or **Java Virtual Machine** is the part of the Java ecosystem that ensures the *write once, run everywhere* principle. 
- It allows the code written on a platform to be run on any other platform.
- JVM was initially designed for Java but as of now it supports more languages such as **Scala**, **Kotlin** and **Groovy**
- The JVM is at its core a Virtual Machine (VM)

### 2. What is a [[Virtual Machine]]?

- A VM is a *virtual representation* of a physical computer on a host machine.
- A single physical machine ca run multiple VMs, each with their own OS and applications.
- The VMs on a physical machine are isolated and act independently of each other.
- The [[Hypervisor]] is in charge of the resource management between the different VMs.
![[VMs distribution - EXC.png]]

### 3. How does the JVM work?

- There are a couple of ways that programming languages process the code:
	- There are *[[Compiled languages|compiled languages]]* that first take the code through a compilation process and transform it in platform specific machine code.
	- There are *[[Interpreted languages | interpreted languages]]* that execute the instructions directly without compiling it using an interpreter.
	- Java uses a combination of both techniques as it first compiles the *.java* files into *.class* files and then the JVM picks-up those classes and executes the code on any version of JVM running on any platform and OS. 
- The JVM acts as a regular VM and creates and isolated space where it can run the Java programs.

### 4. JVM architecture










### N: Resource:
- [[VMs distribution - EXC]]
- [[Virtual Machine]]
- [[Hypervisor]]