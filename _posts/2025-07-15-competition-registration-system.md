---
layout: post
title: 'Competition Registration System'
author: AmcaQt
categories: [Project]
tags: [Project, KKTM PJ, Coding]
render_with_liquid: false
media_subpath: /images/Comp_Registration_Sys
image: 
    path: LOGO-PJ.png
date: 2025-07-15 18:14 +0800
---

![KKTM PJ](LOGO-WEBSITE.jpg){: width="600" height="150" .shadow }
_<https://petalingjaya.kktm.edu.my/>_

After weeks of planning, building, and refining — this is the final form of my **Competition Registration System** developed for **Pertandingan Inovasi KKTM Petaling Jaya** 💻✨

## Project Objective

The main goal was to simplify how participants register for the Innovation Competition. Previously, everything was done through **manual Google Forms or spreadsheets**, which made it hard to track submissions, manage data, and send out confirmations.

So I built a **custom PHP-based web system** with the following core features:

- **Online Form Submission** (with proper input sanitization)
- **Document Uploading** (PDF/DOC/DOCX proof of registration eg: the receipt of the payment)
- **Multiple Email Notification System**:
  - Lecturers receive alerts when someone registers
  - Participants get a confirmation email
- **Admin Panel** for managing the approval process
- **Log System** to keep a record of every registration
- **Cyber-inspired UI** with scroll animations and dark neon aesthetic 🧠💡

---

## How It Works

1. **Participant fills the registration form** with team name, personal info, category, and uploads required docs.
2. **Backend PHP script** processes the data, sanitizes inputs, saves it to a MySQL DB, and stores uploaded files in `/uploads`.
3. It sends an email using **PHPMailer**:
   - One to the lecturer(s)
   - One to each participant in the form
4. A log entry is created for every registration inside `/logs/registration.log`
5. Admin can view, approve, or reject entries via the **Admin Dashboard**.

---

## Tech Stack (Why I Chose Them)

| Tech | Reason |
|------|--------|
| **PHP** | Easy deployment on XAMPP + most Malaysian web hosts use it |
| **MySQL** | Lightweight DB + works seamlessly with PHP |
| **PHPMailer** | Handles secure Gmail SMTP sending easily |
| **Bootstrap + AOS.js** | Quick, responsive styling with scroll animation flair |
| **Jekyll + GitHub Pages** | For documentation like this one 😎 |

---

## Admin Panel Features

> Accessible only to the event coordinator(s)
{: .prompt-info}

- Login system (for now, it's basic session-based)
- List of registered participants
- Approve/Reject system with status
- Pagination and CSV Export functionality
- Sends auto-email upon approval or rejection

---

## Folder Structure Overview

```
Innovation.kktmpj.edu.my/
├── PHPMailer/                  # PHPMailer library
│   ├── src/
│   └── ... 
│
├── img/                        # Images (e.g., logo & banner)
│   └── LOGO-WEBSITE.png
│
├── uploads/                    # Uploaded payment proof documents
│   └── receipt.pdf                
│
├── logs/                       # Log folder for participant logs
│   └── registration.log
│
├── css/                        # CSS Folder for UI and Animation
│   └── style.css
│   └── register.css
│   └── about.css
│   └── competition.css
│
├── index.html                  # Landing page
│
├── Register/                   # Participant registration form
│   └── index.html
│
├── About/                      # Info of KKTM PJ
│   └── index.html
│
├── Competition/                # Info About the Competition
│   └── index.html
│
├── admin/
│   ├── dashboard.php          # Admin panel interface
│   ├── login.php              # Admin login page
│   └── manage_participants.php # View & manage participants
│
├── submit_registration.php     # Form handler with email & logging
│
└── config.php                  # Config file (DO NOT commit!)
```

---

## Real-Life Usage

I presented this to my Lecturer before the deployment: 

![Presenting](AmcaQt-Presenting.png){: width="700" height="400" }
_Presenting to Lecturer_

This project wasn't about coding, this project taught me how to handle backend logic, user experience, email systems, and most importantly, how to build something practical that actually gets used. I had to think about:

- How the admins would **filter and approve entries**
- How to make email responses feel **official and informative**
- How to ensure the system **works on both localhost and live server**
- How to make the site feel **modern and techy**, not just another bland form.

## Future Features

- [ ] Registration Form for Jury
- [ ] Google reCAPTCHA to avoid spam
- [ ] WhatsApp notification bot ( MAYBE ?)
- [ ] Role-based access for multi-admin support

## What I Learned

- Handling file uploads securely
- Building clean, readable email templates
- Using `PHPMailer` for bulk & multiple recipients
- Logging user actions efficiently
- Designing a consistent UI with a "Cyber-IoT" vibe

## Have Inquiries or Question ? 

- 📧 [amcaqt06@gmail.com](amcaqt06@gmail.com)

- 💻 [Github](https://github.com/AmcaQt)

- 📷 [Instagram](https://www.instagram.com/amcaaqt/)

---

Crafted with love, sweat & code, for `Kolej Kemahiran Tinggi Mara Petaling Jaya`

Thanks for reading! Feel free to explore the [source code](https://github.com/AmcaQt/Competition-Registration-System), give it a ⭐ if you found it helpful, or connect with me for any ~~collabs~~ or feedback!
