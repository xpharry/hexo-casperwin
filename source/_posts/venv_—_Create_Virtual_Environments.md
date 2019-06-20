---
title: venv — Create Virtual Environments
date: 2019-06-20 12:00:27
tags:
---

<img src="https://cdn.activestate.com/wp-content/uploads/2018/10/pipenv-better-than-venv-blog-hero-1200x630.jpg" alt="venv" style="height:300px;" align="middle">

# venv — Create Virtual Environments

<!--more-->

To set things up in a virtual environment for isolation is a good way. If you don't already have it install python3's venv module.


    sudo apt-get install python3-venv

Create a venv


    mkdir -p ~/my_venv
    python3 -m venv ~/my_venv

Install rocker


    cd ~/my_venv
    . ~/my_venv/bin/activate
    pip install git+https://github.com/osrf/rocker.git #(just an example)

For any new terminal re activate the venv before trying to use it.


    . ~/my_venv/bin/activate
