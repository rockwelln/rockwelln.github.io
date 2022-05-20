---
title: "Docker Basics"
date: 2022-05-19T10:03:56+02:00
draft: false
---

This is my reference article to introduce docker basic concepts.

## Agenda

1. Overview
    1. Why and What?


## Why do you need docker?

Modern architectures usually involve existing services to handle some responsibilities in the service we want to offer.<br>
We want to use those services because we don't have time to reinvent the wheel and they are robust, cheap/free, standard, ... name it (e.g nginx, haproxy, postgresql, redis, python, etc...).<br>

IT architecture is a moving target full of people with strong opinions. Each of these tools has it's own list of requirements and (even if the team behind them is making a great job to ensure compatibility) there is no guarantee they will work properly together across versions, in the long run. <br>

### Typical dependency issue

For some reason (customer's requirement or OS limitation), you have to use NGINX version A (which is old).<br>
That version requires Openssl version X (which is old too).<br>
So you need python version B which requires Openssl version Y.<br>
This kind of conflict can be resolved with a lot of manual commands, difficult to explain to a new developper and difficult to maintain over time.<br>

### Platform deprecation

Back in time, 8 years ago, Centos was known to be the most stable linux distribution. So we decided to use it as base requirement for our software.<br>
But today Centos is deprecated, only bug fixes and security patches are provided for Centos7 until June, 2024.<br>
(it means, for instance, Centos7 still ships python3.6 which is EOL since 12/2021 without any bug fixes or security patches).<br>

### A solution attempt

We, as software makers, don't want to deal with those recurring problems.<br>
We want to have isolation of service dependencies and abstraction of the operating system.<br>


## Living in a limited environment

### Reading files

You can usually use `cat` for small files or `less` for bigger files

### Patch files

Sometimes we want to patch just a couple of files in a running container.<br>
But because the container is light weight, usually there is no editor (e.g vi, vim or nano) present in the image.<br> 

#### First attempt for text files (sed)

Sed is a very common program in the Linux world and is present in most distro and images (even the slim ones).<br>
Usually scary for beginners using the command line, you can use it as such:

```bash
sed 's/<old string>/<new string>/g' <text_file> > <new_file>
```

For instance

```bash {hl_lines=[7,8]}
# replace the host ip address in some config file
sed 's/host=127.0.0.1/host=1.2.3.4/g' config.yml > newConfig.yml
    
# check the modification applied only where you expect it
diff config.yml newConfig.yml
    
# if you are fine with it, backup and replace the original
cp -v config.yml config.yml.bkp
mv -v newConfig.yml config.yml
```

#### More complex situation

When you need to patch a folder or binaries, `sed` will be no help.<br>

```go {linenos=table,hl_lines=[8,"15-17"],linenostart=199}
// GetTitleFunc returns a func that can be used to transform a string to
// title case.
//
// The supported styles are
//
// - "Go" (strings.Title)
// - "AP" (see https://www.apstylebook.com/)
// - "Chicago" (see https://www.chicagomanualofstyle.org/home.html)
//
// If an unknown or empty style is provided, AP style is what you get.
func GetTitleFunc(style string) func(s string) string {
  switch strings.ToLower(style) {
  case "go":
    return strings.Title
  case "chicago":
    return transform.NewTitleConverter(transform.ChicagoStyle)
  default:
    return transform.NewTitleConverter(transform.APStyle)
  }
}
```
