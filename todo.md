## TODO list:
- fare function compute_normal_derivative
- fare function solve_laplace-beltrami
- cambiare rhs che in step 6 è 0 (credo) e ora hai quello con il coseno
- capire se fare entrambi adaptive o no
- 
 


## Mail



Caro Marco,

quello che ha fatto fino ad ora va bene, ma manca in realtà la parte “cicciosa” che le avevo chiesto. Pensavo ci stesse lavorando sopra, non avevo capito che era arrivato alla fine.

- Dovrebbe risolvere un problema di tipo Laplace-Beltrami sul boundary del dominio (seguendo appunto step-38)
- Dovrebbe usare la soluzione come condizione al contorno per il problema volumetrico (seguendo appunto step-6)

Per ora ha risolto il problema solo sul boundary (usando adaptivity), ma non ha usato questo dato per risolvere il problema sul volume.

In sostanza, il problema che le avevo chiesto di risolvere era questo:

### Problem Statement

Let \(\Omega\) be a bounded domain in \(\mathbb{R}^n\) with a smooth boundary \(\partial \Omega\). We consider the following coupled system of partial differential equations:

1. **Poisson's Equation on \(\Omega\):** (step-6)
  \[
  \begin{cases}
  -\Delta u = f \quad & \text{in } \Omega, \\
  u = g \quad & \text{on } \partial \Omega,
  \end{cases}
  \]
  where \(f\) is a given source term in \(\Omega\) and \(g\) is the Dirichlet boundary data that we will determine from the solution of the Laplace-Beltrami equation on \(\partial \Omega\).

2. **Laplace-Beltrami Equation on \(\partial \Omega\):** (step-38)
  \[
  \Delta_{\partial \Omega} g = h \quad \text{on } \partial \Omega,
  \]
  where \(h\) is a given source term on \(\partial \Omega\) and \(\Delta_{\partial \Omega}\) denotes the Laplace-Beltrami operator on the boundary \(\partial \Omega\).

### Coupling Conditions

To couple these two problems, we impose the following conditions:

- The Dirichlet data \(g\) for the Poisson problem is given by the solution of the Laplace-Beltrami equation:
 \[
 u|_{\partial \Omega} = g.
 \]

- The source term \(h\) for the Laplace-Beltrami equation is given by the normal derivative (flux) of the solution \(u\) of the Poisson problem:
 \[
 h = \frac{\partial u}{\partial n} \bigg|_{\partial \Omega},
 \]
 where \(\frac{\partial u}{\partial n}\) denotes the outward normal derivative of \(u\) on \(\partial \Omega\), cioè credo \(\frac{\partial u}{\partial n} = N \cdot \nabla u\) dove $N$ è il vettore uscente unitario normale al bordo.

### Non-trivial Solution

To ensure a non-trivial solution, we pick a non zero f, for example:

- \(f(x) = \sin(\pi x_1)\) for \(x \in \Omega\).

### Summary of the Coupled Problem

1. **Poisson's Equation on \(\Omega\):**
  \[
  \begin{cases}
  -\Delta u = \sin(\pi x_1) \quad & \text{in } \Omega, \\
  u = g \quad & \text{on } \partial \Omega.
  \end{cases}
  \]

2. **Laplace-Beltrami Equation on \(\partial \Omega\):**
  \[
  \Delta_{\partial \Omega} g = \frac{\partial u}{\partial n} \bigg|_{\partial \Omega} \quad \text{on } \partial \Omega.
  \]

### Solution Procedure

Start with zero g = gbar.

1. **Solve the Poisson problem** to find \(u\) (with dirichlet data gbar)
2. **Compute the normal derivative** of \(u\) on \(\partial \Omega\) to define \(h\).
3. **Solve the Laplace-Beltrami equation** on \(\partial \Omega\) to find \(g\).
4. Check difference between g and gbar: if smaller than tol -> stop

Iterate.

Per iniziare, cominci a scrivere il problema di Poisson, e ad accoppiarlo con quello di Laplace-Beltrami, con un dato h fittizio (ovvero consideri i due problemi accoppiati solo in un senso: g è la condizione al contorno per u, ma u non influenza g).

Mi dica fin dove riesce ad arrivare.

Ps: le serviranno le funzioni

DoFTools::extract_boundary_dofs

DoFTools::map_dofs_to_support_points

In sostanza, una volta inizializzati i due dof handler, deve identificare i gradi di libertà sul bordo con quelli sulla superficie.


DoFTools::map_dofs_to_support_points costruisce una mappa dai gradi di libertà ai punti di supporto (la utilizzi due volte, una per il volume, e una per la superficie)

DoFTools::extract_boundary_dofs le permette di filtrare solo i punti di supporto del volume che le servono.

fa un paio di loop per identificare quando due pti di supporto su volume e superfice sono vicini tra loro a meno di eps, e quando lo sono, identifica i gradi di libertà del volume con quelli della superficie.

Una volta fatto questo, può usare g[i] come dato al contorno per u[map[i]], ovvero

constraints.add_line(map[i])

constraints.set_inhomogeinity(map[i], g[i])

Con questo dovrebbe riuscire a fare la prima parte.