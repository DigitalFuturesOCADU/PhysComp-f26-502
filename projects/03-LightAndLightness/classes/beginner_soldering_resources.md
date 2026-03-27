# Beginner Soldering Resources

A curated list of videos, tutorials, and guides for learning to solder.

---

## 📹 Video Series

### PACE Worldwide – Basic Soldering Lessons (9-part series)
The gold standard of soldering instruction. This classic nine-episode series covers everything from what solder really is to how to correctly solder integrated circuits. Highly recommended even if you already know the basics.
- [Full playlist on YouTube](https://www.youtube.com/playlist?list=PL926EC0F1F93C1837)
- [Hosted on PACE's website](https://paceworldwide.com/video/basic-soldering-lesson-1-solder-flux)

### EEVblog – Soldering Tutorial (YouTube)
A well-regarded practical walkthrough from electronics educator Dave Jones:
- [EEVblog #180 – Soldering Tutorial Part 1](https://www.youtube.com/watch?v=J5Sb21qbpEQ)
- [EEVblog #183 – Soldering Tutorial Part 2](https://www.youtube.com/watch?v=fYz5nIHH0iY)

---

## 📖 Written Guides

### SparkFun – How to Solder: Through-Hole Soldering
Covers the basics of through-hole soldering, discusses the tools needed, goes over techniques for proper soldering, and also covers rework — with tips and tricks that make fixing any piece of electronics easier.
🔗 https://learn.sparkfun.com/tutorials/how-to-solder-through-hole-soldering/all

### Makerspaces – How to Solder: A Complete Beginners Guide
Outlines the basics of soldering irons, soldering stations, types of solder, desoldering, and safety tips — and includes a free 17-page PDF ebook.
🔗 https://www.makerspaces.com/how-to-solder/

### iFixit – Soldering 101: A Beginner's Guide
Covers the bare minimum you need to get started, including tools and gear, safety precautions, and basic soldering skills. Recommends starting with through-hole components as they're easier to see and handle, making them ideal for learning.
🔗 https://www.ifixit.com/News/6864/how-to-solder

### Build Electronic Circuits – How to Solder
A practical step-by-step guide covering soldering two wires together and soldering components on a circuit board, with good detail on what makes a good vs. bad (cold) solder joint.
🔗 https://www.build-electronic-circuits.com/how-to-solder/

### Jameco Electronics – Beginner's Guide to Soldering Basics
Provides an overview of the tools, supplies, and techniques required for those new to soldering electronic components.
🔗 https://www.jameco.com/Jameco/workshop/learning-center/soldering-basics.html

---

## 🛠️ Interactive / Step-by-Step

### Stanford University – Intro to Soldering
Designed as a structured intro: after reading the material, students take an online quiz, then schedule hands-on instruction, and finally take a hands-on exam before being certified to use soldering equipment. Great model for a classroom setting.
🔗 https://sites.google.com/stanford.edu/soldering-internal/learning

### Instructables – How to Solder (with Pictures)
Focuses on soldering for the beginner and explains how to solder a variety of components using a few different techniques.
🔗 https://www.instructables.com/How-to-solder/

---

# Introduction to Soldering: Course Outline

---

## 1. What Is Soldering? *(Big picture — 1 paragraph)*

Soldering is the process of using heat and a metal alloy (solder) to create a permanent, electrically conductive bond between two metal surfaces. It's the fundamental skill behind almost all electronics — if you've ever looked at the back of a circuit board, every component is held in place with a solder joint. It's not difficult, but it does reward patience and practice.

---

## 2. What You'll Need *(Materials overview)*

A short, non-intimidating list with brief explanations of *why* each item matters:

- **Soldering iron** — the heat source; a basic 25–40W temperature-controlled station is ideal for beginners
- **Solder** — the metal alloy that forms the joint; lead-free rosin-core is recommended for classroom use
- **Helping hands / third-hand tool** — holds your work so both hands are free
- **Tip cleaner (brass wool or damp sponge)** — keeps the iron tip healthy and effective
- **Wire strippers** — for preparing wire ends
- **Flush cutters** — for trimming leads on protoboard
- **Ventilation** — fumes from flux are irritating; work near a fan or open window

> *"You don't need expensive tools to start. A $20–30 soldering station will handle everything in this class."*

---

## 3. How Soldering Works *(Conceptual, not procedural — 1 paragraph)*

The key insight beginners miss: you're not melting solder *onto* a surface — you're heating the *joint itself* so the solder flows into it. A good joint happens when both surfaces are hot enough to accept the solder. A bad joint (called a "cold joint") happens when only the solder melts but the surfaces underneath don't — it looks dull and grainy and won't conduct reliably. This one idea fixes most beginner mistakes before they happen.

---

## 4. Step-by-Step: The Core Technique

1. **Tin the tip** — melt a small amount of solder onto the clean iron tip before you start
2. **Prepare your surfaces** — strip wire ends; if using protoboard, insert component leads through holes
3. **Heat the joint, not the solder** — hold the iron tip against both surfaces for 2–3 seconds
4. **Feed solder into the joint** — touch solder to the *joint* (not the iron); let it flow
5. **Remove solder, then iron** — pull the solder wire away first, then the iron
6. **Don't move it** — let the joint cool undisturbed for a few seconds
7. **Inspect** — a good joint is shiny and cone-shaped; dull or blobby means try again
8. **Trim leads** (protoboard) — use flush cutters to clip excess wire close to the joint

---

## 5. In This Class: Two Specific Applications

### A. Adding wires to light sensor leads
*Why:* Bare component legs are short and fragile. Adding wire extensions lets you position sensors wherever you need them.
- Strip ~5mm of insulation from each wire end
- Tin both the wire and the sensor lead separately before joining them
- Twist together and reheat to flow solder through both — this is called a "lap joint"
- Cover with heat shrink or electrical tape for strain relief

### B. Soldering onto protoboard
*Why:* Protoboard (also called perfboard or stripboard) lets you build permanent circuits beyond the breadboard stage.
- Insert component leads or wires through the holes
- Bend leads slightly on the back side so components don't fall out while you work
- Heat pad + lead together, feed solder in, aim for a small volcano shape
- Each joint should be isolated — check for unintended bridges between pads

---

## 6. What to Watch For: Common Problems

| Problem | Likely Cause |
|---|---|
| Dull, grainy joint | Cold joint — didn't heat the surfaces enough |
| Solder won't stick | Dirty/oxidized surface, or tip not tinned |
| Solder bridges two pads | Too much solder, or iron dragged sideways |
| Component shifted | Moved before the joint fully cooled |
| Iron tip goes black | Tip needs cleaning and re-tinning |

---

## 7. Going Further *(Reference section)*

Once the basics feel comfortable, here's where to go next:

- **Improve your technique** — watch the [PACE Basic Soldering series](https://www.youtube.com/playlist?list=PL926EC0F1F93C1837) (9 short videos); widely considered the best instructional content ever made on the subject
- **Deeper written reference** — [SparkFun's Through-Hole Soldering Guide](https://learn.sparkfun.com/tutorials/how-to-solder-through-hole-soldering/all) covers everything including desoldering and rework
- **Repairs and troubleshooting** — [iFixit's Soldering 101](https://www.ifixit.com/News/6864/how-to-solder) is great for understanding how to fix joints that didn't work
- **Quick general reference** — [Makerspaces.com guide](https://www.makerspaces.com/how-to-solder/) with free PDF download
- **Surface-mount soldering (SMD)** — the next level after through-hole; [Build Electronic Circuits](https://www.build-electronic-circuits.com/how-to-solder/) has a good intro to that progression

---

## Notes on Tone and Format

- Lead with the **two concrete tasks** students will do that day (wire extensions, protoboard) — it grounds the abstract parts immediately
- The **cold joint concept** in Section 3 is worth spending real time on verbally; it's the single idea that changes how beginners think about the process
- The **troubleshooting table** works well as a physical handout at the bench
- Skip surface-mount for now — through-hole and wire work is plenty for a first session and builds real confidence
