# TSP Genetic Algorithm with Adaptive Mutation

This solution uses a **memetic algorithm** approach that combines:
- **Greedy + Random Initialization**: Hybrid population with up to 50% nearest neighbor tours and random permutations for diversity
- **Adaptive Mutation**: Changes exploration/exploitation balance based on evolution phase
- **2-opt Local Search**: Periodic refinement of elite solutions to eliminate tour crossings

The algorithm automatically adjusts parameters based on problem size to balance solution quality and computational time.

---

## Algorithm Structure

### 1. **Initialization** 
Hybrid population strategy:

#### `calculate_cost(tour, distance_matrix) -> float`
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
Creates hybrid population with **up to 50% greedy tours**.
- **Parameters**:
  - `distance_matrix`: Problem instance
  - `pop_size`: Population size
- **Returns**: List of tour arrays

---

### 2. **Genetic Operators**

#### `tournament_selection(population, fitness, tournament_size=3) -> ndarray`
Selects parent by running mini-competitions.
- **Process**: Randomly picks `tournament_size` individuals, returns best one
- **Returns**: Copy of selected tour

#### `order_crossover(parent1, parent2) -> ndarray`
Preserves relative city positions from both parents.
- **Process**: Copy random segment from parent1 and fill remaining positions with parent2 cities in order
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

Automatically tunes parameters based on problem size:

| Cities | Generations | Pop Size | Elite | 2-opt Freq | 2-opt Iter |
|--------|-------------|----------|-------|------------|------------|
| ≤50    | 300         | 100      | 10    | every 10   | 50         |
| ≤200   | 400         | 120      | 12    | every 15   | 50         |
| ≤500   | 500         | 150      | 10    | every 20   | 50         |
| >500   | 500         | 150      | 10    | every 30   | 10         |

Larger problems use less frequent/intensive local search to balance quality and runtime.


