# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **documentation-only repository** for the WordPress.com REST API. It contains structured markdown files that document the API's functionality, authentication methods, and usage examples. This is not a code repository - there are no build processes, tests, or executable code.

It contains the works to revamp the WordPress.com REST API documentation pages at https://developer.wordpress.com/docs/api/. 

## Repository Structure

The documentation is organized as numbered markdown files that represent different sections:

- `1-home.md` - Main landing page and overview
- `2-getting-started-with-the-api.md` - Getting started guide with authentication and base URL structure
- `3-namespaces.md` - API namespaces and versions
- `4-reference.md` - API reference documentation
- `5-oauth2-authentication.md` - OAuth2 authentication details
- `6-wordpress-com-connect.md` - WordPress.com Connect information
- `7-using-the-rest-api-from-js-and-the-browser-cors.md` - Browser usage and CORS
- `__8-rest-api-urls-by-site-type.md` - URL structure by site type (deprecated, prefixed with __)
- `wordpress-api-authentication-options.md` - Authentication options overview

## Content Guidelines

When working with this documentation:

1. **Maintain consistency** in URL examples - use `https://public-api.wordpress.com` as the base
2. **Use real examples** where possible - site ID `241031857` is used throughout as a standard example
3. **Keep authentication examples current** - focus on Personal Access Tokens and OAuth2
4. **Follow the established structure** - each file has a specific purpose and cross-references others
5. **Preserve markdown formatting** - code blocks, headers, and links are carefully structured

## Key Documentation Patterns

- **Base URL Structure**: All examples use `https://public-api.wordpress.com/{namespace}/{version}/...`
- **Site-specific URLs**: Include site ID like `/sites/241031857/posts`
- **Authentication**: Show both Bearer token and OAuth2 examples
- **Code Examples**: Include curl, JavaScript, and PHP examples where appropriate
- **Cross-references**: Link between related documentation sections

## TODO Comments

The documentation contains TODO comments that indicate areas needing completion or clarification. When making changes, check for and address relevant TODOs.

## No Build/Test Commands

This repository contains only documentation files. There are no:
- Package managers (no package.json, composer.json, etc.)
- Build scripts
- Test suites
- Linting tools
- CI/CD configurations

All work involves editing markdown files directly.