---
layout: default
title: Debugging and fixing pyinstaller hanging on Github workflow (Github actions)
feed: true
date: 2026-01-14
---

# Debugging and fixing pyinstaller hanging on Github workflow (Github actions)
{: .no_toc }


{: .fs-6 .fw-300 }

{% if page.date %}
  <time datetime="{{ page.date | date_to_xmlschema }}">
    {{ page.date | date: "%Y-%m-%d" }}
  </time>
{% endif %}

# Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


---

# The problem

Sometimes when you try to build a Python app in Github actions, pyinstaller gets stuck indefinitely. Even enabling --log-level=DEBUG wouldn't produce anything useful. However, pyinstaller runs successfully on your local machine (if pyinstaller also gets stuck on your local machine, you can also read the following guide).

If you see **Looking for dynamic library** roughly at the end of the Github action output, it typically means that certain packages are causing the problem.

The issue is described in this [page](https://github.com/pyinstaller/pyinstaller/issues/8396).

Some common problematic packages include PySimpleGUI, and python-magic

# How to find out what went wrong and fix it
To find out which package caused the hang, you could follow the instructions in [page](https://github.com/pyinstaller/pyinstaller/issues/8396). If your pyinstaller hangs on your local machine, it is pretty straightforward to modify the local code of pyisntaller. However, if the issue is present only on Github action, it can be a little more complicated to modify the code.

For your convenience, here are several lines of code you can put into your Github workflow file. The code is written for windows, but transforming into a Linux version is straightforward.
```
$sitePackages = python -c "import site; print(site.getsitepackages()[0])"
$targetFile = Join-Path $sitePackages "Lib/site-packages/PyInstaller/building/build_main.py"
$pattern = 'child.call\(import_library, package\)'
$insertion ='logger.info("attempting to import: %r", package)'
$originalLine= '                    child.call(import_library, package)'
$replacement = "$insertion`n$originalLine"
(Get-Content $targetFile) -replace $pattern, $replacement | Set-Content $targetFile
```

Then, run pyinstaller, and you can see exactly which package went wrong. Replace the buggy package or do whatever necessary.


