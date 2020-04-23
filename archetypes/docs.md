---
title: "{{ replace (replaceRE "[0-9]{4}-[0-9]{2}-[0-9]{2}-(.+)" "$1" .Name | title) "-" " " }}"
description: ""
date: {{ .Date }}
weight: 20
draft: true
bref: ""
toc: true
---
