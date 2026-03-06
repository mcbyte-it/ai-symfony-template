# New project template

Using this project as a template, create for me a collection of MD files inside @docs/template that instruct claude code how to create a similar project from scratch, a boiler plate.

**when executing prompts in this file, only create and edit files in the @docs/template/ folder, you should NOT create any project files, not execute any command other than reading files mentioned here to take their content for the prompts to be written.**

**DO NOT TOUCH OR CHANGE this file, template.md, when creating new files keep this untouched.**

all md file instructures should be clear for claude, use best practices in prompt engineering, think well, use best secure and follow standards for development

# Files to create

## 1-NEW_PROJECT.md

- a main NEW_PROJECT.md thats asks few questions then execute some commands, and also execute sub md files.
- the NEW_PROJECT should create claude.md like the one in this project @CLAUDE.md, showing global project description (put a placeholder, project testing), tech stack, communication, architecture and folder structure, memory.md file that is updated with each step, a prompt_history.md file to save each prompt used, changelog.
- telling claude that when a new feature is requested (and git is used in the project) then create a new branch for the new feature, develop everything in the new branch.
- docs/README.md (the the current @docs/README.md ) which indicates how to develop for this porject, php 8.4, PSR12.

## 2-BACKEND.md

- main file BACKEND.md is about the general framework: latest symfony 7.4 running on php 8.4, project that uses symfony packages doctrine 2.18, dbal 4.4, flex, rate limiter, twig, apache-pack, maker bundle.
- the symfony project should always be secure, use csrf, xss, sql injection, rate limiter.
  configure the database to use MariaDB 12.2, and make a config setting TABLE_PREFIX to always be used when creating any table.
- the project will mostly always have users and security for login: create with migrations a users table (which will also have TABLE_PREFIX) that includes basic users table things (id, date created, date updated, is_active) which will have username, email, hash password (secure hashing, argn2id with secure settings), user name (real name), roles, failed_attemps, locked until, must_change_pw. and next to this table also a table for remember me tokens.
- ask the user if they want also RBAC or RLS for this project, and procede as answered. for RBAC create table to assign roles to users
- create a command to create a user in the database, ask the user for info (or command arguments) and create user in DB

## 3-GIT.md

- the project will be in a git repo, so prepare gitignore file for symfony, precommit config like the one here @.pre-commit-config.yaml
- the pre commit should also install php-cs-fixer with config like in @.php-cs-fixer-dist.php.
- update README to tell the user how to use GIT and what will git do (pre-commit for leaks, php-cs-fixer), and git commands needed.

## 4-FRONTEND.md

- for FRONTEND.md, make is just a place holder, by default use bootstrap 5.x (or whatever latest), and icons from FontAwesome 7.x (latest). Should use modern web design, use the front end skill if available. user need to edit it at a later point.

## 5-TESTING.md

- TESTING.md: ask the user if they want to implement testing configuration, in case of yes then import to the project composer the phpunit test and phpstan, and create relative unit testing config files and folders, and write in claude.md that it should create unit test files for everything it creates.

## 6-CI Workflows

- CI.md which prepares a sample gitea\workflows ci.yml code, using the current @ci.yml file as sample. should do checkout, check twig lint, stan, unit test with coverage, call sonarqube, and depending on the project.

## 7-PRETTIER.md

- PRETTIER.md, which installs and configure prettier for the project, take the default config from the current @.prettierrc and @.prettierignore , and .vscode/settings.json to tell prettier extension how to auto formatr code.

## plan-progress.md

- WIP...
- create a file with steps needed to create the project, including symfony initialization, users table, testing etc etc
- each step will have a checkbox [ ], and tell claude to update this file when plan features are beging implemented

create also a file tempate 0-PROJECT_DESCRIPTION.md
where the user fills in what the new project is going to be about, put in some sections, requirements.md, and fill it with sample data.
such as project brief, this is an invoicing app that will have clients, services, invoices with statuses. need to have many users, need to have reporting etc etc...
make a template file with some fake info and a clear notice to the user _MODIFY THIS FILE AS REQUIRED_.
add a section that automatically answers the questions later, will this need testing, what kind of front end, CI needed, etc.
