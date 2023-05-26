The following tool configurations should be enabled in Jenkins to properly work
with the Declarative Pipeline multibranch jobs:

# JDK

A JDK with a proper `JAVA_HOME` should be provided.

# Git

A Git installation should be provided. If `git` is available in the `PATH`, 
then it is enough to point to `git`.

# SonarQube Scanner

Add installations of SonarQube scanner:

- Name: SonarQube Scanner 3, install automatically from Maven Central, version: 
  SonarQube Scanner 3.1.0.1141
- Name: SonarQube Scanner 4, install automatically from Maven Central, version: 
  SonarQube Scanner 4.7.0.2747

# Ant

Add an installation of Ant:

- Name: Ant 1.10.1, install automatically from Apache, version: 1.10.1

# Python

Add installation of Python:

- Name: System-CPython-3, path: `/usr/bin/python3.8`
