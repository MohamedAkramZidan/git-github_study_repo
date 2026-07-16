# Git and GitHub Study Repository

This repository contains study notes and practical exercises for understanding Git architecture and daily workflow.

## Core Concepts

### The 3-Tree Architecture
Git manages files across three distinct areas:
* Working Tree (WT): The local directory where files are actively edited.
* Staging Area (Index): The preparation zone where files are added before committing.
* Repository (Repo / .git): The local database storing permanent snapshots.

### File States
Files in Git transition through different states:
* Untracked (U): New files not yet watched by Git.
* Tracked: Files Git is monitoring.
  * Unmodified: No changes since the last commit.
  * Modified (M): File has been changed.
    * M (Red): Modified but not in the index (not staged).
    * M (Green): Modified and added to the index (staged).

### Git Objects
Git generates a unique SHA-1 hash for everything and stores them in the .git/objects folder. The primary objects are:
* Blob: Stores raw file content and metadata.
* Tree: Stores directory structure and groups of blobs.
* Commit: Stores snapshot metadata (author, date, message) and points to a tree.
* Tagged Annotation: Used to mark specific commits.

## Standard Command Workflow

* git init
  Initializes a new repository by creating the .git directory structure.

* git status
  Shows the current state of the working directory and staging area.

* git status -s
  Provides a condensed, short status summary.
  Example output:
  M  file.txt

* git ls-files
  Lists all files currently tracked in the repository.
  Example output:
  README.md
  file.txt

* git add .
  Moves all modified and untracked files into the Staging Area.

* git commit -m "message"
  Saves the staging area as a permanent commit in the repository.

* git log
  Displays the commit history, including the SHA-1 hash, author details, date, and message.
