---
theme: seriph
background: https://images.unsplash.com/photo-1626958390943-a70309376444?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2070&q=80
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
transition: slide-left
title: Reverse Engineering RGB Keyboard Backlights
mdc: true
hideInToc: true
---

# Reverse Engineering RGB Keyboard Backlights

Rishit Bansal

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
hideInToc: true
---

# 👤 Hi, I'm Rishit Bansal!

- Founding Team / Dev @ Dyte (https://dyte.io)
- Contributor/Maintainer of CoCo (https://coco.build/)
- I play CTFs with csictf, dytesec
- 2nd time at Nullcon, participant last year
  - Placed 1st along with dytesec at hardware CTF

<br>
<br>

[https://www.linkedin.com/in/bansal-rishit/](https://www.linkedin.com/in/bansal-rishit/)

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<!--
Here is another comment.
-->

---
layout: center
hideInToc: true
---

# 🎯 Goals

- Reverse Engineer a Windows Service which interacts with keyboard backlight firmware
- Understand the system protocols involved in interacting with device firmware
- Re-implement the same functionnality on the Linux Kernel
- Bonus: Discover new functionality possible in hardware!

<br />
<br />

<!-- ### Chapters
<Toc maxDepth="1"></Toc> -->

---
level: 1
layout: center
---

## Chapter 1
# Reverse Engineering the HP Light Studio Application

---
level: 2
layout: default
---

# HP Omen Light Studio Application 

Introduction on what the Omen Light Studio Application is


---
level: 2
layout: default
---

# Locating the Service

Introduction on what the Omen Light Studio Application is


---
level: 2
layout: default
---

# Introduction to ILSpy

Introduction to dotpeek, and passing the service files to dotpeek

---
level: 2
layout: default
---

# Decompiling the Executable and DLL Files (Locating the main() function)

Showing the main() function in the code

---
level: 1
layout: center
---

# Chapter 2: Understanding WMI and ACPI

---
level: 1
layout: center
---

# Chapter 3: Developing WMI Drivers on the Linux Kernel