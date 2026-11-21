# 📘 CS Notebook Guide

**Plain-language explanations of every concept in a Computer Science degree (Universiti Malaya-style syllabus), with examples, code, and everyday analogies.**

Each guide turns dense lecture slides into something you can actually read the night before (or, better, the week before 😉) an exam. Every topic has:

- **In plain words:** what the concept *is*, without jargon
- **🧠 An analogy:** a real-life comparison so it sticks
- **💻 Examples / code:** worked examples, Java, Python, SQL, VHDL, or Cisco IOS where it fits
- **✅ A quick self-check** at the end of each guide

---

## 📚 Index

### Year 1 · Semester 1
| Course | Guide | What you'll learn |
|---|---|---|
| WIX1001 Computing Mathematics | [CM-computing-mathematics.md](year1-sem1/CM-computing-mathematics.md) | sets, logic, proofs, functions, relations, matrices, counting, graphs, trees |
| WIX1003 Computer System Organization | [CSO-computer-system-organization.md](year1-sem1/CSO-computer-system-organization.md) | number systems, 2's complement, Boolean algebra, K-maps, logic gates, adders, flip-flops, counters |
| WIX1002 Fundamentals of Programming | [FOP-fundamentals-of-programming.md](year1-sem1/FOP-fundamentals-of-programming.md) | Java basics, selection, loops, arrays, methods, file I/O, OOP, inheritance, polymorphism, exceptions |
| WIX2001 Thinking & Communication Skills | [TCS-thinking-and-communication-skills.md](year1-sem1/TCS-thinking-and-communication-skills.md) | verbal and non-verbal communication, presentations, critical thinking, arguments, fallacies, problem solving |

### Year 1 · Semester 2
| Course | Guide | What you'll learn |
|---|---|---|
| WIA1003 Computer System Architecture | [CSA-computer-system-architecture.md](year1-sem2/CSA-computer-system-architecture.md) | performance, instruction cycle, interrupts, buses, cache, memory, RAID, I/O, ISA, CISC vs RISC, pipelining |
| WIA1002 Data Structures | [DS-data-structures.md](year1-sem2/DS-data-structures.md) | generics, ADTs, linked lists, stacks, queues, graphs, recursion, sorting, BSTs, Big-O |
| WIA1006 Machine Learning | [ML-machine-learning.md](year1-sem2/ML-machine-learning.md) | pre-processing, regression, decision trees, neural networks, HMMs, clustering, PCA, SVM, GMM/EM, RL, genetic algorithms |
| WIA1005 Network Technology Foundation | [NTF-network-technology-foundation.md](year1-sem2/NTF-network-technology-foundation.md) | OSI/TCP-IP, Ethernet, switching, ARP, IP routing, TCP/UDP, subnetting, IPv6, VLANs, DHCP |

### Year 2 · Semester 1
| Course | Guide | What you'll learn |
|---|---|---|
| WIA2008 Advanced Network Technology | [ANT-advanced-network-technology.md](year2-sem1/ANT-advanced-network-technology.md) | STP, EtherChannel, LAN security, OSPF, ACLs, NAT, VPN/IPsec, SNMP/syslog, SDN, network security |
| Digital Design with HDL (VHDL) | [DDHDL-digital-design-vhdl.md](year2-sem1/DDHDL-digital-design-vhdl.md) | VHDL entity/architecture, dataflow/structural/behavioural styles, sequential logic, testbenches, FSMs, ASM, FPGAs |
| WIA2001 Database | [DB-database.md](year2-sem1/DB-database.md) | relational model, relational algebra, ER modelling, normalization, SQL, PL/SQL, transactions and concurrency |
| Systems Analysis & Design | [SAD-systems-analysis-and-design.md](year2-sem1/SAD-systems-analysis-and-design.md) | SDLC, waterfall vs agile, project selection and management, feasibility, requirements, DFDs, decision tables |
| WIA2003 Probability & Statistics | [PS-probability-and-statistics.md](year2-sem1/PS-probability-and-statistics.md) | probability, Bayes, binomial/Poisson/normal, descriptive stats, hypothesis testing, t-tests, regression, non-parametric tests |

---

## 🔗 How the courses connect

```
CM (logic, sets, graphs) ──────► DS (graphs, trees, recursion) ──────► ML (algorithms)
   │                                    ▲                                 ▲
   └──► CSO (Boolean algebra) ──► DDHDL (VHDL circuits)                   │
            │                                                             │
            └──► CSA (CPU, cache, pipelining)                             │
FOP (Java) ────────────────────► DS (Java data structures)                │
NTF (networking basics) ───────► ANT (advanced networking)                │
SAD (system design) ───────────► DB (database design)                     │
CM (counting) ─────────────────► P&S (probability & statistics) ──────────┘
TCS (critical thinking) ───────► every presentation, report & group project 🙂
```

## 🧭 How to study with these guides
1. **Skim the guide first** to get the big picture, *then* read your lecture slides. They'll make much more sense.
2. **Cover the analogy and explain the concept out loud** in your own words. If you can teach it, you know it.
3. **Type out and run the code examples.** Change something and predict what happens.
4. **Do the self-check** at the end of each guide, then attempt past-year questions.

## 🤝 Contributing
Found a mistake or have a better analogy? Open an issue or a pull request. Contributions are welcome!

---

*These guides summarise and explain course topics in original words for learning purposes. Always check your official lecture notes and course outline for exactly what's examinable.*
