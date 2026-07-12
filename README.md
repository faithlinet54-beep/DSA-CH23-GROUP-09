# DSA-CH23-GROUP-09
Smart Parking Slot Allocation System
=====================================

BIT 4105 - Advanced Data Structures and Algorithms Group Project
Theme E: Elevator / Valet Parking OO System, Variant E3 (Parking allocation by size rules + fast slot lookup using a hash table and a heap)

Group members:
Pheneas Maina, Faith Akinyi, Vialbreal Rewain, Claire Kihwaga, Titus Peter


Problem Statement
------------------
Most parking lots just assign slots first-come, first-served, without thinking about how big the vehicle actually is. That causes two problems we kept running into when we were discussing this: a big slot meant for an SUV gets taken by a motorcycle, and then an actual SUV shows up later and has nowhere to fit even though the lot technically has empty space. On top of that, if someone needs to check where a specific car is parked, there's usually no fast way to do it besides walking the lot or checking a paper log.

For this project we designed a system that assigns each arriving vehicle to the best free slot in its own size category (motorcycle, compact/sedan, SUV, truck/van) and can find any vehicle's current slot almost instantly using its ticket ID or plate number. The brief for this project doesn't require a working app, so what's in this folder is mainly the written report, plus a small script we put together to check that the hash table + heap logic we describe actually works the way we say it does.


Features
--------
- Gives a vehicle the nearest/best-fit free slot in its category, not just any free slot
- Keeps motorcycle, compact, SUV and truck slots separate
- Can find an active transaction by ticket ID or plate in close to constant time
- Puts a vehicle on a waiting list if its category is full, and allocates it a slot automatically once one is freed
- Frees a slot as soon as a vehicle exits
- Can show how many slots are free and how many vehicles are waiting per category
- Design was checked against 1,000, 10,000+ and 1,000,000+ transaction levels (see the full report for this part)


Architecture
------------
This is roughly how the pieces fit together:

    Vehicle arrives / leaves (attendant app or gate sensor)
                    |
                    v
        -------------------------
             Allocation Engine
        -------------------------
        - Hash table: ticket ID -> transaction   (fast lookup)
        - One min-heap per category: free slots ordered by distance (best-fit allocation)
        - One waiting queue per category
                    |
                    v
              Slot Registry (array of all slot objects)
                    |
                    v
        Boom gate  /  availability display  /  manager reports

When a vehicle comes in, the allocation engine takes the best slot off that category's heap, saves the transaction in the hash table, and updates the slot registry. When a vehicle leaves, we look its transaction up in the hash table, push the slot back onto the heap, and if someone is waiting in that category's queue, they get the slot right away.

We did talk about using a balanced tree instead of a heap at one point, mainly because it would let us do range-type queries like "show me every free slot within 20 metres of the entrance." We decided a heap was enough for what this project needs, but the tree option is written up as an alternative in the main report.


How to Run
----------
You just need Python 3.8 or newer, nothing else to install.

    cd smart-parking-system
    python3 parking_simulation.py

The script sets up a small compact-car section, runs a few vehicle entries, one lookup, and one exit, and prints out what happens at each step.


Sample Input and Output
------------------------
Input used for this demo run:

    3 compact slots: C1 (5m from entrance), C2 (12m), C3 (20m)
    Entries: T101/KDA 101A, T102/KDB 202B, T103/KDC 303C, T104/KDD 404D
    Lookup: T101, T999 (doesn't exist)
    Exit: T101

What we got when we ran it:

    === Vehicle entries ===
    [ALLOCATED] KDA 101A (T101) -> slot C1 (compact, 5m from entrance)
    [ALLOCATED] KDB 202B (T102) -> slot C2 (compact, 12m from entrance)
    [ALLOCATED] KDC 303C (T103) -> slot C3 (compact, 20m from entrance)
    [NO SLOT] KDD 404D (T104) has no free compact slot - added to waiting list.

    === Ticket lookup ===
    [FOUND] T101 -> plate KDA 101A, slot C1, entered 18:32:09
    [NOT FOUND] No active transaction for ticket T999.

    === Vehicle exit ===
    [RELEASED] Slot C1 (compact) is now free.
               [ALLOCATED] KDD 404D (T104) -> slot C1 (compact, 5m from entrance)

    === Category availability ===
    Category    Free Slots   Waiting
    motorcycle  0            0
    compact     0            0
    suv         0            0
    truck       0            0

Basically it does what we said it would: the heap hands out the closest slot first (C1, then C2, then C3), the hash table finds T101 straight away, and T104, who had been waiting, gets a slot the moment C1 opens up.


Team Member Roles
------------------
Member 1 - Pheneas Maina - Hash Table / Map. Handles fast lookup and update of active transactions by ticket ID or plate number.

Member 2 - Faith Akinyi - Heap / Priority Queue. Handles best-fit allocation and release of slots within each size category.

Member 3 - Vialbreal Rewain - Array / Dynamic Array. Handles the base slot registry, indexed by slot number.

Member 4 - Claire Kihwaga - Balanced Tree (AVL). Worked on the alternative design for ordered/range-based slot queries and the trade-off discussion.

Member 5 - Titus Peter - Queue (FIFO). Handles the waiting list for vehicles that arrive when their category is full.


Files in This Folder
---------------------
- Parking_System_Group_Project_Report.docx - the full written report (problem justification, Chapter 3/23 analysis, complexity tables, transaction-level evaluation)
- parking_simulation.py - the demo script mentioned above


References
----------
Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2022). Introduction to Algorithms (4th ed.). MIT Press.
BIT 4105 Course Notes and Reader, Chapters 3 and 23.
Anthropic. (2026). Claude (Sonnet 5) [Large language model]. https://claude.ai
