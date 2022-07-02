---
categories: OS
date: "2022-06-16T00:00:00Z"
tags: ['OS', 'process', 'nohup']
title: 'nohup은 왜 쓰는걸까 ?'
---
 
# nohup
: `nohup`은 이름 그대로, hup(hang up) 을 방지하는 POSIX 명령어 중 하나입니다. man 페이지에 `nohup`을 찾아보면 다음과 같이 설명합니다. 

> The nohup utility invokes utility with its arguments and at this time sets the signal
SIGHUP to be ignored.  If the standard output is a terminal, the standard output is
appended to the file nohup.out in the current directory.  If standard error is a
terminal, it is directed to the same place as the standard output.

man 페이지에서 이야기하는 `SIGHUP`은 `signal hang up`의 약자로, 
