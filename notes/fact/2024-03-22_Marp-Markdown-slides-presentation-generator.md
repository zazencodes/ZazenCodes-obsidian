---
date: 2024-03-22
tags:
    - fact
hubs:
    - "[[markdown]]"
urls:
    - https://marp.app/
---

# Marp Markdown slides presentation generator

e.g.

Render below with `marp  --allow-local-files --theme assets/dracula.css --html file.md`

```markdown
---
theme: dracula
_class: lead
paginate: true
footer: patreon.com/ZazenCodes
---

<style scoped>section { font-size: 30px; }</style>

![bg right:40%](assets/zc_logo_2024_03_lg_white.png)

# ZazenCodes Notes

## BigQuery for Big Data Engineers 2021

Instructor: J Garg
Platform: Udemy

---

<style>
    section { font-size: 24px; }
    li { color: white !important; }
    code { font-size: 18px !important; }
    .columns {
      display: grid;
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 1rem;
    }
</style>

## Section 1: Introduction to GCP

- Why GCP?
    - Separation of compute and storage
    - Highly available
    - Ease of scalability

- Services overview
    - From most to least customizable
        - Compute Engine is barebones VM / server
        - App Engine is abstraction of Compute Engine
        - Cloud Functions are serverless


---

```




## Multi-column layout for slide

e.g.

Render below with `marp --html file.md`

```markdown
---
marp: true
style: |
  .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
---

# Multi columns in Marp slide

<div class="columns">
<div>

## Column 1

Lorem ipsum dolor sit amet consectetur adipisicing elit. Voluptas eveniet, corporis commodi vitae accusamus obcaecati dolor corrupti eaque id numquam officia velit sapiente incidunt dolores provident laboriosam praesentium nobis culpa.

</div>
<div>

## Column 2

Tempore ad exercitationem necessitatibus nulla, optio distinctio illo non similique? Laborum dolor odio, ipsam incidunt corrupti quia nemo quo exercitationem adipisci quidem nesciunt deserunt repellendus inventore deleniti reprehenderit at earum.

</div>
</div>

```
