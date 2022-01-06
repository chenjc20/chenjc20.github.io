---
layout: homepagepost
title: THU FASTsys Research Group
ename: Guangyan Zhang
cname: 张广艳
role: Associate Professor, CCF senior member
org1: · Tsinghua University
org2: · Department of Computer Science and Technology
org3: · FASTsys Research Group
img: assets/img/team/zgy.jpg
room: Room 8-208, East Main Building, Tsinghua University, Beijing, China
email: gyzh@tsinghua.edu.cn
---
## Introduction
* **Guangyan Zhang** is an associate professor in the Department of Computer Science and Technology, where he joined since July 2008. He obtained his Ph.D degree from Tsinghua University under the guidance of **Prof. Weimin Zheng** and **Prof. Jiwu Shu**. Before that, he received the bachelor's and master's degrees in computer science from Jilin University in 2000 and 2003. 
* He is a Professional Member of the ACM, a member of **the CCF technical committee of information storage technology**, a Communication Member of **CCF Task Force on Big Data**.

## Education
* Bachelor of Computer Science, Jilin University, China, 2000
* Master of Computer Science, Jilin University, China, 2003
* Ph.D. in Computer Science, Tsinghua University, China, 2008

## Research
--- His current research focuses on Big Data Computing, Storage Systems, and Distributed Systems, especially in:
* Cloud Storage
* RAID and Erasure Codes
* Flash and PCM Storage
* Storage Virtualization
* Distributed File Systems
* Deep Learning
* Graph Computing
* Stream Computing
* Benchmarking

## Courses
--- He teaches three courses, one of which is for graduate students, and the others are for undergraduate students.
* CS 70240013: Advanced Computer Architecture, for Ph.D and master students,
* CS 40240443: Computer Architecture, for undergraduate students, and
* CS 30240522: Program Design and Training, for undergraduate students.

## Publications
<div class="container">
      <div class="row">
        <div class="col-lg-12">
          <ul class="timeline">
            {%- for event in site.data.sitetext.timeline.events -%}
            <li class="timeline-inverted">
                <h4 style="display: inline;">{{ event.title }}</h4>
                <h6 style="display: inline;margin-left: 10px;">
                      <a href="{{ event.link }}" style="color:#2291CD; font-size: 15px;text-decoration: none;">[PDF]</a>
                </h6>
                  <div class="text-muted">{{ event.auth | markdownify }}</div>
                  <div class="text-muted">{{ event.desc | markdownify }}</div>
            </li>
		  {% endfor %}	
          </ul>
        </div>
      </div>
    </div>
