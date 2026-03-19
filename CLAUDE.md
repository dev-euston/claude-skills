# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A collection of custom Claude Code skills. Each skill is a directory at the root containing a single `SKILL.md` with YAML frontmatter (`name`, `description`) and instructions in the body.

## Conventions

- Skills that depend on another skill will say so explicitly — always honour those references.
- File-producing skills use `<output_dir>/` as a placeholder for the output path.

## Workflow

The user adds new skills manually, then runs Claude to sync the repository. When syncing:

1. Update `README.md` to reflect all current skills.
2. Commit all changes with a message prefixed `[CS-0]` (or a ticket number if provided). Use an imperative summary, e.g. `[CS-0] add commit skill`.
