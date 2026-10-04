# KEN3170 – Plant Tissue Simulations

## Assignment

### 1. Pathogen infection over 2 hours

The infected region starts on the left side and slowly spreads further into the tissue. As the infection spreads, the cells close to it become more deformed and irregular because their walls get weaker.

**T = 0 min**

![T = 0](images/infection_t0.png)

**T = 30 min**

![T = 30](images/infection_t30.png)

**T = 60 min**

![T = 60](images/infection_t60.png)

**T = 90 min**

![T = 90](images/infection_t90.png)

**T = 120 min**

![T = 120](images/infection_t120.png)

### 2. Cell wall stiffness

A cell's wall starts with stiffness 3. As its chemical level increases, the stiffness is reduced using `stiffness = 3 - chemical level`. The chemical effect is capped, so the stiffness can only decrease to about 1.8. The pathogen cells (`CellType 2`) are different. They do not weaken their own walls. Instead, they keep stiffness at 3, grow larger, and divide when they reach the division threshold.

### 3. Cell-to-cell transport and feedback

The diffusion coefficient is `0.00001 / stiffness`. So when there is more chemical, the wall gets less stiff, and when the stiffness is lower the chemical diffuses faster. This makes the chemical spread to more cells, which can weaken more walls. It is positive feedback because the chemical helps itself spread more.

### 4. Effect of `rel_cell_div_threshold`

When `rel_cell_div_threshold = 3`, the pathogen has to grow much bigger before it can divide, so the population expands more slowly. When `rel_cell_div_threshold = 1`, it reaches the division condition much sooner, so it divides faster and the pathogen population grows more quickly. In the pictures, by T = 120 the pathogens look almost the same size, but after 5 hours we can clearly see that threshold 1 has produced more pathogen cells.

**`rel_cell_div_threshold = 3`**

![Threshold 3](images/threshold_3_t0_t120.png)

**`rel_cell_div_threshold = 1`**

![Threshold 1](images/threshold_1_t0_t120.png)

**Comparison at T = 300 min**

![Threshold comparison at T = 300](images/threshold_comparison_t300.png)

### 5. Cell neighbours

In the other models we worked with (auxin transport and auxin growth), a cell's neighbours never change. The only way a cell gets a new neighbour is through division. This matches real plants, where cells are glued together by the middle lamella and cannot slide past each other.

In the infection model, neighbours can change during the simulation. In `CellHouseKeeping`, healthy cells get `SetCellVeto(true)`, but cells weakened by the pathogen chemical get `SetCellVeto(false)`. In `mesh.cpp`, wall elements can only be reconfigured for cells without a veto. This means wall segments can be moved from one cell to the neighbouring cell. Once the walls are weakened, the borders between cells are no longer fixed. The pathogen keeps growing (`EnlargeTargetArea(2)`) and dividing, so it can push into the weakened tissue and squeeze in between plant cells. That way it can get new neighbours it did not touch at the start, similar to how fungal hyphae grow into real plant tissue.

In our run (see [section 1](#1-pathogen-infection-over-2-hours)), the cells next to the pathogen turn purple as the chemical reaches them, which means their walls are weakened and their veto is off. Between [T = 0](images/infection_t0.png) and [T = 120](images/infection_t120.png), the purple region grows to about two columns of cells, and the pathogen grows and divides into two cells while pushing against the weakened left edge.

Another difference is that wall stiffness is stored per wall element and per cell side. A wall shared by two cells can have a different stiffness on each side, and `getLengthAndStiffness()` takes the average of both sides when calculating diffusion.

### 6. Plant defense

The defense would go in `CellHouseKeeping`, inside the "cell wall weakening happens here" block, for plant cells only (`CellType != 2`). The stiffness is set again for every cell at every step, so the defense check has to come before the weakening rule. Otherwise the weakening would overwrite it.

```
// new parameters
defense_threshold   // chemical level that switches on the defense (higher than 0.1)
defense_stiffness   // wall stiffness of a defended cell (higher than 3)

CellHouseKeeping(c):
    if c is pathogen (type 2):
        grow and divide as before              // unchanged

    set base wall element length as before     // unchanged

    patho_chem_level = min(Chemical(0) / 0.5, 1.2)

    if c is not pathogen:
        if patho_chem_level > defense_threshold:
            // defense: the cell senses a lot of pathogen chemical
            for each wall element of c:
                stiffness = defense_stiffness
            SetCellVeto(true)                  // walls can no longer be reconfigured
        else if patho_chem_level > 0.1:
            // weakening (original rule)
            for each wall element of c:
                stiffness = 3 - patho_chem_level
            SetCellVeto(false)
        else:
            // healthy (original rule)
            for each wall element of c:
                stiffness = 3
            SetCellVeto(true)
```

Optional extension: with the version above, a cell loses its defense as soon as the chemical drops below the threshold again. To make the defense last, the unused second chemical (`Chemical(1)`) could be used as a defense marker:

```
CellDynamics(c):
    if c is not pathogen and Chemical(0) is above defense_threshold:
        build up Chemical(1)
    else:
        slowly degrade Chemical(1)

CellHouseKeeping(c):
    use "Chemical(1) > marker_threshold" as the defense condition

SetCellColor(c):
    give defended cells their own colour so the barrier is visible
```

This adds **negative feedback**. The original loop is positive: more chemical → lower stiffness → higher diffusion (`0.00001 / stiffness`) → more spread. The defense turns the middle step around: more chemical → higher stiffness → lower diffusion → slower spread into and through that cell. A rise in chemical now triggers a response that limits further rise.

The stiffer walls also have a mechanical effect: they weigh more in the wall length part of the Hamiltonian, so they resist being stretched by the growing pathogen. Setting the veto back to true also stops the pathogen from pushing in between cells. We would expect the infection to slow down or stop, with a ring of stiff cells forming around it. Because the defense only switches on above the threshold, cells at the front would still be weakened for a short time before they defend. So the positive feedback dominates at low chemical levels and the negative feedback takes over at high levels.
