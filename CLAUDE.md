# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A GitBook-based hands-on educational guide: "Containers with Podman." It covers containers from introductory concepts through advanced topics like pods, networking, volumes, and OpenShift. The content targets Podman 5.x with Netavark networking, Aardvark-DNS, pasta rootless networking, and modern base images (UBI 9, Alpine 3.21). There is no application code, build system, or test suite — the repo is entirely Markdown content and image assets.

## Structure

- `SUMMARY.md` — GitBook table of contents; defines chapter ordering and navigation. **Must be updated** when adding/renaming/moving pages.
- `README.md` — Landing page for the guide.
- `chapter-*/` — Each directory is one chapter, containing theory pages and `practice-*.md` labs.
- `extra-openshift/` — Supplementary content (Podman Compose, Docker Compose, Kubernetes/OpenShift).
- `.gitbook/assets/` — Screenshots and images referenced from Markdown pages.

## Conventions

- Chapter directories use the naming pattern `chapter-N-topic-name/`.
- Practice/lab files are prefixed with `practice-`.
- Theory or intro files within a chapter are often named `untitled.md` (GitBook default).
- Image references point to `.gitbook/assets/` using relative paths.
- Podman uses `Containerfile` as the preferred file name (though `Dockerfile` is also supported).
- Command outputs use `$` prompt (not `[root@servera ~]#`).
- Content is authored by Swapnil Jain; some theory is sourced from external documentation.
