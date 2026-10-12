# AI for Agile: examples

The files shown in the *AI for Agile* talk series, as they are in the project they come from.

*AI for Agile* is a series of screencast talks for everyone on a software team: project managers,
engineers, tech leads and QA. Its claim is that AI made coding many times faster, that coding is just
one role on a software team, and that the way to make the whole team faster is to adapt your software
development process to the strengths and the limits of an AI on the team.

Each episode after the intro takes one step of that process and shows the files one real team uses for
it: the standing prompts, the skills, and the documents they produce. This repository holds those files.

## What this is

These are one team's files, from one project: an online board game called Variational Go, built by a
small team with Claude Code. They are published here so that you can read them in more detail than a
talk allows.

They are not a template. Every team is different, and a process copied from another team will not fit
yours. Read these, see what they do and why, and then write your own.

## Layout

The layout mirrors the project the files come from, so what you see on screen in a talk is what you
find here.

- `CLAUDE.md`: the project-wide standing prompt, loaded on every command.
- `.claude/skills/`: the skills, one directory each. A skill is a written process for one kind of work,
  loaded only while that work is happening.
- `doc/`: example documents the skills produce, such as functional and technical specifications.

Files arrive here as the episodes that show them are released.

## Licence

MIT. See `LICENSE`.
