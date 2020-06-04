---
title: "{{ replace (replaceRE "[0-9]{4}-[0-9]{2}-[0-9]{2}-(.+)" "$1" .Name | title) "-" " " }}"
slug: "{{ replaceRE "[0-9]{4}-[0-9]{2}-[0-9]{2}-(.+)" "$1" .Name }}
description: ""
date: {{ .Date }}
weight: 20
draft: true
bref: ""
toc: true
---
