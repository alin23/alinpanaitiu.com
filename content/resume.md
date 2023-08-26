---
title: Résumé - Alin Panaitiu
date: 2019-12-16T10:01:37+02:00
draft: false
show_title_as_headline: false
layout: resume

name: Alin Panaitiu
profession: Fullstack Developer
description: |-
  Professional developer with 9+ years of experience in creating robust services with Python, Go, Crystal and your frontend framework of choice and 6 years of experience in mobile and desktop app development using Swift and C.
location: Brașov, Romania
email: alin@panaitiu.com
phone: +40 763 728 495
birthdate: 1994-10-23
birthplace: Matca, Romania
nationality: Romanian
pdf: /resume.pdf
links:
  - url: https://www.producthunt.com/@alin23/made
    domain: Product Hunt
    color: "#DA552F"
  - url: https://github.com/alin23
    domain: Github
    color: "#181717"
skills:
  - name: Swift
    level: 100
  - name: Python
    level: 100
  - name: PostgreSQL
    level: 100
  - name: Crystal
    level: 90
  - name: React
    level: 83
  - name: GraphQL
    level: 83
  - name: Information Security
    level: 83
  - name: Docker, K8s, AWS
    level: 83
  - name: Go
    level: 66
  - name: Rust
    level: 50
languages:
  - name: Romanian
    level: 100
  - name: English
    level: 90
  - name: Italian
    level: 30
jobs:
  - company: Comfy
    position: Backend Developer and Devops Engineer
    logo: comfy.svg
    location: Remote
    startdate: September 2019
    enddate: March 2021
    url: https://www.comfyapp.com/
    domain: comfyapp.com
    stack:
      - name: Python
        level: 100
      - name: Go
        level: 100
      - name: gRPC
        level: 90
      - name: Kubernetes
        level: 70
      - name: Javascript
        level: 40
      - name: Mapbox
        level: 20
      - name: Vue
        level: 30
      - name: PostgreSQL
        level: 30
      - name: PostGIS
        level: 80
    story: |-
      *   Improving the backend infrastructure by implementing microservice related features:
          *   Dynamic centralised configuration service that replaces the need to keep and modify static file configuration in every project
      *   Go gRPC gateway for providing both an RPC and a REST interface to other services
      *   Microservices written in Python and Go for functionalities like:
          *   Public transport departure times based on office location
          *   Available parking spots near the office
          *   Food menu for nearby restaurants
      * Migrating old services to asyncio
      * Migrating Python 2 code to Python 3
      * Unified authentication using Auth0 and Azure AD
      * Map tile service for the campus and buildings map
        * Implemented using PostGIS on the backend and Mapbox on the frontend

  - company: Arcanabio
    position: Fullstack Developer and Devops Engineer
    logo: arcanabio.png
    location: Remote
    startdate: September 2018
    enddate: July 2019
    url: https://www.arcanabio.com
    domain: arcanabio.com
    stack:
      - name: Python
        level: 100
      - name: JS/Coffeescript
        level: 100
      - name: ReactJS
        level: 100
      - name: NodeJS
        level: 40
      - name: PostgreSQL
        level: 100
      - name: Docker Swarm
        level: 70
    story: |-
      *   Python backend for API and DNA sequence analysis using NCBI tools
          *   The code was fully developed in an async manner using asyncio, GraphQL and asynchronous Redis queues
      *   ReactJS frontend using Next.js for routing and server-side rendering
          *   Redux and Hooks were used for state management and Redux Sagas for side-effects
          *   I helped speed up development by using a direct connection to the PostgreSQL database using a GraphQL middleware and handling the security with the Row-Level Security feature of PostgreSQL
      *   Infrastructure management using Docker Swarm

  - company: iMedicare (now Amplicare)
    position: Python Backend Developer
    logo: amplicare.png
    location: Remote
    startdate: September 2016
    enddate: August 2017
    url: https://amplicare.com
    domain: amplicare.com
    stack:
      - name: Python
        level: 100
      - name: PostgreSQL
        level: 80
      - name: Keras
        level: 60
    story: |-
      *   Maintaining a Flask backend API + workers and scrapers
          *   I worked on both the customer facing web app and the app internaly used by the Sales and Support teams
      *   Developing a Medication Adherence detection algorithm
          *   Heavy use of numpy + pandas for keeping the runtime code fast and memory usage as low as possible
      *   Machine Learning algorithm for improving detection of patient insurance plans

  - company: Bitdefender
    position: Malware Researcher
    logo: bitdefender.svg
    location: Iași, Romania
    startdate: March 2014
    enddate: June 2016
    url: https://www.bitdefender.com
    domain: bitdefender.com
    stack:
      - name: Python
        level: 100
      - name: x86 Assembly
        level: 80
      - name: C++ (WinAPI)
        level: 40
      - name: Java
        level: 50
    story: |-
      *   Malware analysis using various techniques of reverse engineering
      *   Automating parts of the workflow using Python
      *   Automated Java malware detection using Aspect Oriented Programming:
          *   Mostly using AspectJ for hooking into code at runtime and decrypting the malware code or gathering info about the C&C servers it uses
          *   I automated the system using Python and the VirtualBox APIs so that malware samples can be run and analyzed as soon as they are found and provide a fast response in the antivirus solution

images:
  - https://img.panaitiu.com/_/og/plain/https%3A%2F%2Falinpanaitiu.com%2Fimages%2Fresume-screenshot.png@webp
image:
  url: alin.jpg
  topcolor: rgba(233, 10, 49, 0.3)
  bottomcolor: rgba(15, 230, 106, 0.3)
education:
  - institution: Faculty of Computer Science
    location: '"Alexandru Ioan Cuza" University of Iași'
    startdate: October 2013
    enddate: June 2016
projects:
  - name: Lunar
    attributions: Developer and designer
    startdate: March 2018
    enddate: Present
    links:
      - url: https://lunar.fyi
        domain: lunar.fyi
      - url: https://github.com/alin23/lunar
        domain: github.com
      - url: https://www.producthunt.com/posts/lunar-5-2
        domain: producthunt.com
    description: |-
      Lunar adds adaptive brightness for external monitors, by making use of the built-in light sensor of the Macbook/iMac, computing sunrise/noon/sunset times for the current location and adding hotkeys for manually adjusting the adaptive algorithm to suit your environment.
    stack:
      - Swift
      - C
      - Crystal
  - name: The low-tech guys
    attributions: Creator, developer, designer
    startdate: November 2021
    enddate: Present
    links:
      - url: https://lowtechguys.com
        domain: lowtechguys.com
    description: |-
      A small macOS and iOS app studio, with the goal of creating apps with utility in mind. We aim to solve every day annoyances in working with Apple devices.
    stack:
      - Swift
      - Crystal
      - Python
      - C

  - name: Noiseblend
    attributions: Developer and designer
    startdate: October 2017
    enddate: Present
    links:
      - url: https://www.noiseblend.com
        domain: noiseblend.com
      - url: https://github.com/Noiseblend
        domain: Github
      - url: https://www.producthunt.com/posts/noiseblend
        domain: Product Hunt
      - url: https://www.producthunt.com/posts/noiseblend-for-alexa
        domain: Alexa Skill
    description: |-
      The goal of Noiseblend is to help Spotify users dive into the enormity of the music collection that Spotify provides, and bring to surface the songs that best match their taste.
    stack:
      - Python
      - ReactJS
      - PostgreSQL
      - InfluxDB
      - Docker Swarm
---

