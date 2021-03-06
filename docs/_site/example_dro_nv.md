<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

### A multi-item newsvendor problem considering the Wasserstein ambiguity set

In this example, we consider the multi-item newsvendor problem discussed in the paper [Robust stochastic optimization made easy with RSOME](http://www.optimization-online.org/DB_FILE/2017/06/6055.pdf). This newsvendor problem determines the order quantity \\(w_i\\) of each of the \\(I\\) products under a total budget \\(d\\). The unit selling price and ordering cost for each product item are denoted by \\(p_i\\) and \\(c_i\\), respectively. The uncertain demand of each product item is denoted by the random variable \\(\tilde{z}_i\\). Once the demand realizes, the selling quantity \\(y_i\\) is expressed as \\(\min\{x_i, z_i\}\\), and the newsvendor problem can be written as the following distributionally robust optimization model,

$$
\begin{align}
\min~& -\pmb{p}^T\pmb{x} + \sup\limits_{\mathbb{P}\in\mathcal{F}(\theta)}\mathbb{E}_{\mathbb{P}}\left[\pmb{p}^T\pmb{y}(\tilde{s}, \tilde{\pmb{z}}, \tilde{u})\right] && \\
\text{s.t.}~&\pmb{y}(s, \pmb{z}, u) \geq \pmb{x} - \pmb{z}, && \forall \pmb{z} \in \mathcal{Z} \\
& \pmb{y}(s, \pmb{z}, u) \geq \pmb{0} && \forall \pmb{z} \in \mathcal{Z}\\
& \pmb{c}^T\pmb{x} = d, ~ \pmb{x} \geq \pmb{0}
\end{align}
$$    

with \\(s\\) the scenario index, and \\(u\\) the auxiliary random variable. The recourse decision \\(y_i\\) is approximated by the decision rule \\(y_i(s, \pmb{z}, u)\\) which adapts to different scenarios \\(s\\) and is affinely adaptive to the random variables \\(\pmb{z}\\) and the auxiliary variable \\(u\\). The model maximizes the worst-case expectation over a Wasserstein ambiguity set \\(\mathcal{F}\\), expressed as follows.

$$
\begin{align}
\mathcal{F}(\theta) = \left\{
\mathbb{P} \in \mathcal{P}_0\left(\mathbb{R}^I\times\mathbb{R}\times [S]\right) \left|
\begin{array}
~(\tilde{\pmb{z}}, \tilde{u}, \tilde{s}) \in \mathbb{P} &\\
\mathbb{E}_{\mathbb{P}}\left[\tilde{u} | \tilde{s} \in [S]\right] \leq \theta & \\
\mathbb{P}\left[\left.\pmb{0}\leq \pmb{z} \leq \overline{\pmb{z}}, \|\pmb{z} - \hat{\pmb{z}}_s\|_2 \leq u ~\right| \tilde{s} = s\right] = 1, & \forall s \in [S] \\
\mathbb{P}\left[\tilde{s} = s\right] = \frac{1}{S} &
\end{array}
\right.
\right\}
\end{align}
$$

with \\(\theta \geq 0\\) the parameter capturing the distance between the distribution \\(\mathbb{P}\\) and the empirical distribution \\(\hat{\pmb{z}}\\). In this numerical experiment, parameters of the model and the ambiguity set are specified as follows:

- The number of products: \\(I=2\\);
- Sample size: \\(S=50\\);
- The unit cost of each product: \\(c_i=1\\)
- The unit price of each product: \\(p_i\\) is randomly generatd from a uniform distribution on \\([1, 5]\\).
- The upper bound of demand: the array \\(\overline{\pmb{z}}\\) is randomly generated from the uniform distribution on \\([0, 100]^I\\);
- The sample data of demands: the array \\(\hat{\pmb{z}}\in\mathbb{R}^{S\times I}\\) is randomly generated from the uniform distribution on \\([\pmb{0}, \overline{\pmb{u}}]\\);
- The budget \\(d=50 I\\);
- The Wasserstein metric parameter \\(\theta=0.01 \min\\{\overline{\pmb{z}}\\}\\).

The RSOME code for implementing the model above is given as follows.

```python
from rsome import dro
from rsome import norm
from rsome import E
from rsome import grb_solver as grb
import numpy as np
import numpy.random as rd

# Model and ambiguity set parameters
I = 2
S = 50
c = np.ones(I)
d = 50 * I
p = 1 + 4*rd.rand(I)
zbar = 100 * rd.rand(I)
zhat = zbar * rd.rand(S, I)
theta = 0.01 * zbar.min()

# Modeling with RSOME
model = dro.Model(S)                        # Create a DRO model with S scenarios
z = model.rvar(I)                           # Random demand z
u = model.rvar()                            # Auxiliary random variable

fset = model.ambiguity()                    #
for s in range(S):
    fset[s].suppset(0 <= z, z <= zbar,
                    norm(z - zhat[s]) <= u) # Define the support for each scenario
fset.exptset(E(u) <= theta)                 # The Wasserstein metric constraint
pr = model.p                                # An array of scenario probabilities
fset.probset(pr == 1/S)                     # Support of scenario probabilities

x = model.dvar(I)                           # Define first-stage decisions
y = model.dvar(I)                           # Define decision rule variables
y.adapt(z)                                  # y affinely adapts to z
y.adapt(u)                                  # y affinely adapts to u
for s in range(S):
    y.adapt(s)                              # y adapts to each scenario s

model.minsup(-p@x + E(p@y), fset)           # Worst-case expectation over fset
model.st(y >= 0)                            # Constraints
model.st(y >= x - z)                        # Constraints
model.st(x >= 0)                            # Constraints
model.st(c@x == d)                          # Constraints

model.solve(grb)                            # Solve the model by Gurobi
```

```
Being solved by Gurobi...
Solution status: 2
Running time: 0.0271s
```
