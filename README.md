# TSP Genetic Algorithm with Adaptive Mutation

A hybrid genetic algorithm implementation for solving the Traveling Salesman Problem (TSP), combining greedy initialization, adaptive mutation strategies, and 2-opt local search optimization.

## 🎯 Overview

This solution uses a **memetic algorithm** approach that combines:
- **Greedy + Random Initialization**: 50% nearest neighbor tours, 50% random permutations for population diversity
- **Adaptive Mutation**: Changes exploration/exploitation balance based on evolution phase
- **2-opt Local Search**: Periodic refinement of elite solutions to eliminate tour crossings
- **Numba JIT Compilation**: Accelerates critical functions for large-scale problems

The algorithm automatically adjusts parameters based on problem size to balance solution quality and computational time.

---

## 📊 Algorithm Structure

### 1. **Initialization** 
Hybrid population strategy balancing quality and diversity:

#### `calculate_cost(tour, distance_matrix) -> float`
**JIT-compiled** function that computes total tour distance.
- **Parameters**: 
  - `tour`: City visit sequence
  - `distance_matrix`: Pairwise city distances
- **Returns**: Total tour cost including return to start

#### `nearest_neighbor(distance_matrix, start_city) -> ndarray`
Greedy heuristic constructing tours by always visiting the nearest unvisited city.
- **Parameters**:
  - `distance_matrix`: Problem instance
  - `start_city`: Starting point (default: 0)
- **Returns**: Complete tour as city indices array

#### `initialize_population(distance_matrix, pop_size) -> List[ndarray]`
Creates hybrid population with **50% greedy + 50% random** tours.
- **Parameters**:
  - `distance_matrix`: Problem instance
  - `pop_size`: Population size
- **Strategy**:
  - Generates `min(n_cities, pop_size//2)` greedy tours with uniform city spacing
  - Fills remainder with random permutations
- **Returns**: List of tour arrays

---

### 2. **Genetic Operators**

#### `tournament_selection(population, fitness, tournament_size=3) -> ndarray`
Selects parent by running mini-competitions.
- **Process**: Randomly picks `tournament_size` individuals, returns best one
- **Returns**: Copy of selected tour

#### `order_crossover(parent1, parent2) -> ndarray`
Preserves relative city positions from both parents.
- **Process**: 
  1. Copy random segment from parent1
  2. Fill remaining positions with parent2 cities in order
- **Returns**: Valid offspring tour

#### Mutation Operators

**`swap_mutation(tour) -> ndarray`** - Randomly exchanges two cities

**`inversion_mutation(tour) -> ndarray`** - Reverses a random tour segment

**`insertion_mutation(tour) -> ndarray`** - Removes a city and reinserts it elsewhere

#### `adaptive_mutation(tour, mutation_rate, generation, max_generations) -> ndarray`
Changes mutation strategy based on evolution phase:
- **Early (0-30%)**: Inversion + Insertion (exploration)
- **Mid (30-70%)**: Mixed strategy (balanced)
- **Late (70-100%)**: Swap + Inversion (refinement)

---

### 3. **Local Search**

#### `two_opt(tour, distance_matrix, max_iter=100) -> ndarray`
**JIT-compiled** local optimizer that eliminates edge crossings.
- **Process**: Iteratively reverses tour segments until no improvement found
- **Parameters**:
  - `max_iter`: Maximum optimization iterations
- **Returns**: Improved tour

---

### 4. **Main Algorithm**

#### `genetic_algorithm_tsp(...) -> Tuple[ndarray, float, dict]`
Core evolutionary loop with the following workflow:

**Each Generation**:
1. **Evaluate** fitness for all individuals
2. **Update** global best solution
3. **Elitism**: Keep top `elite_size` individuals
4. **Breed** offspring via tournament selection + order crossover
5. **Mutate** offspring with adaptive strategy
6. **Local Search**: Apply 2-opt to elite solutions every `two_opt_freq` generations

**Parameters**:
- `pop_size`: Population size
- `elite_size`: Number of best individuals preserved
- `mutation_rate`: Probability of mutation (default: 0.2)
- `max_generations`: Evolution budget
- `two_opt_freq`: How often to apply local search
- `two_opt_max_iter`: 2-opt iteration budget

**Returns**: `(best_tour, best_cost, history)`

---

### 5. **Adaptive Configuration**

#### `solve_problem(problem_file) -> dict`
Automatically tunes parameters based on problem size:

| Cities | Generations | Pop Size | Elite | 2-opt Freq | 2-opt Iter |
|--------|-------------|----------|-------|------------|------------|
| ≤50    | 300         | 100      | 10    | every 10   | 100        |
| ≤200   | 400         | 120      | 12    | every 15   | 100        |
| ≤500   | 500         | 150      | 10    | every 20   | 100        |
| >500   | 400         | 150      | 10    | every 25   | 30         |

**Rationale**: Larger problems use less frequent/intensive local search to balance quality and runtime.

---

## 📈 Visualization

#### `plot_evolution(history, problem_name)`
Displays algorithm progress with:
- **Cost curve**: Best solution cost over generations (dark green line)
- **Phase backgrounds**: Color-coded mutation strategy phases
  - 🔵 **Blue (0-30%)**: Exploration phase with Inversion + Insertion
  - 🟠 **Orange (30-70%)**: Mixed phase with all mutation operators
  - 🔴 **Red (70-100%)**: Refinement phase with Swap + Inversion
- **Improvement markers**: Orange dots indicate when better solutions are found

---

## 🚀 Performance Features

1. **Numba JIT Compilation**: 
   - `calculate_cost()` and `two_opt()` compiled to machine code with caching
   - 10-100x speedup for large problems

2. **Balanced Initialization**:
   - Avoids premature convergence (pure greedy)
   - Maintains genetic diversity (50% random)

3. **Adaptive 2-opt**:
   - More aggressive for small problems (frequent, deep search)
   - Lighter for large problems (infrequent, shallow search)

---

## 📦 Usage

```python
# Solve single problem
result = solve_problem('problem_g_100.npy')

# Batch processing
problems = ['problem_g_10.npy', 'problem_g_20.npy', ...]
results = [solve_problem(p) for p in problems]
```

**Output**:
```
============================================================
Problem: problem_g_100 (100 cities)
Time: 8.45s
Best Solution: 1234.56
============================================================
```
