---
icon: fa-solid fa-magnifying-glass
order: 4
---

I work for the BYU [Cybersecurity Research Lab](https://csrl.byu.edu/) (CSRL) on a student team doing security research on IoT devices. We focus on finding vulnerabilities in these products, which we then document, file for CVEs, and submit to the company in the hopes of a patch being released.

## Vilo
From January through May 2024, I worked with my team to hack into a SOHO router from a company by the name of [Vilo](https://store.viloliving.com/). They are a relatively new startup and make only two routers, which appear to be very similar in terms of design and functionality. We spent the semester researching this product, dumping firmware, and finding vulnerabilities. Ultimately, we ended up finding 9 CVEs, most of which are of critical severity. After contacting the company about our results, they deployed a patch in August 2024.

Our entire research report can be viewed in our [Github repository](https://github.com/byu-cybersecurity-research/vilo). We gave a talk about it at DEF CON 32, titled "Finding 0days in Vilo Home Routers" ([recording](https://youtu.be/IyInMgXj4k4?si=SIJoCS57tjtDEBrt)) as well as at SAINTCON 2024, with a few updates ([recording](https://youtu.be/TPEaW8JXOAI?si=nhoQv48ARQu_ejEn)).

Our experience with this project proves that even as university students (and mostly undergraduates at the time) with limited time, resources, and relevant experience outside of CTF competitions, our skills were enough to find multiple high-impact vulnerabilities in just a few months. The state of security in IoT devices is atrocious and our research only highlights that.
