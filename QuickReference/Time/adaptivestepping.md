= Time adaptation strategy for incompressible Navier Stokes equations and convection-diffusion equations
:author: Christophe Prud'homme
:toc:

The following strategy can be adapted to any first order in time PDE. We focus here on the Navier-Stokes equations to illustrate the scheme but it can be easily adapted to other PDE.

We present here a 2nd order in time adaptive strategy based on the Crank-Nicolson (CN) scheme coupled with Adam-Bashforth order2 (AB2) to adapt time steps.
The particularity is that the schme works with the discrete acceleration rather than the velocity in order to get a more accurate/stable schemes

== Notations

Denote $$\Omega \subset \mathbb{R}^3$$ the computation domain and $$\partial \Omega$$ its boundary. $$\partial \Omega = \partial \Omega_D \cup \partial \Omega_N $$ which correspond to the Dirichlet and Neumann boundaries

We wish to solve the NS equations on $$[0,T]$$, we denote $$\{t_n\}_{n=0,\ldots,N}$$ the interval discretisation points and $$\{k_n\}_{n=0,\ldots,N}$$. We denote $$(\mathbf{u}^n,p^n, \mathbf{d}^n)$$ the velocity, pressure and discrete acceleration at time $$n$$.

 - $$\mu$$ : dynamic viscosity
 - $$\rho$$ : density
 - $$\mathbf{n}$$: the unit outward normal to $$\partial \Omega$$
 - $$(\cdot,\cdot)$$ : the $$L^2$$ scalar product in $$\Omega$$
 - $$(\cdot,\cdot)_{\partial \Omega}$$ : the $$L^2$$ scalar product on $$\partial \Omega$$
 - $$\mathbf{f}$$ the volumic force density

 - $$\mathbf{u}^n = \mathbf{g}_D(t^{n}) \text{ on } \partial \Omega_D$$
 - $$\sigma(\mathbf{u}^n,p^n)\cdot \mathbf{n} = (-p^{n} I + 2\mu D(\mathbf{u}^{n}) ) \cdot \mathbf{n} = \mathbf{g}_N(t^{n}) \text{ on } \partial \Omega_N$$
 
== Navier Stokes equations

We start by writing the NS equations using the CN scheme.
We first extrapolate the convection velocity using a 2nd order formula
$$
\mathbf{w}^{n+1} = \left(1+\frac{k_{n+1}}{k_n}\right)\mathbf{u}^n-\frac{k_{n+1}}{k_n} \mathbf{u}^{n-1}
$$

then we find $$(\mathbf{u}^{n+1},p^{n+1})$$ such that for all $$(\mathbf{v},q)$$
$$
\begin{split}
(2 \rho \mathbf{u}^{n+1}/k_{n+1},v) + ( \mu \nabla \mathbf{u}^{n+1}, \nabla v)  + ( \rho  \mathbf{w}^{n+1} \cdot \nabla \mathbf{u}^{n+1} , v)  \\
- ( p^{n+1}, \nabla \cdot v )  - (  \nabla \cdot \mathbf{u}^{n+1}, q ) + \\ ( \sigma(\mathbf{u}^{n+1}, p^{n+1}) \cdot \mathbf{n},v)_{\partial \Omega} = (2\rho \mathbf{u}^{n}/k_{n+1},v) + (\frac{\partial \mathbf{u}^n}{\partial t}, v) +( \mathbf{f}, v)
\end{split}
$$

where
$$
\frac{\partial \mathbf{u}^n}{\partial t}:=\mu \Delta \mathbf{u}^n - u^n\cdot \nabla \mathbf{u}^n - \nabla p^n
$$
which reads
$$
\begin{split}
(2 \rho \mathbf{u}^{n+1}/k_{n+1},v) + ( \mu \nabla \mathbf{u}^{n+1}, \nabla v)+
 ( \rho  \mathbf{w}^{n+1} \cdot \nabla \mathbf{u}^{n+1} , v) - ( p^{n+1}, \nabla \cdot v )  -  \\
 (  \nabla \cdot \mathbf{u}^{n+1}, q )  = (2\rho \mathbf{u}^{n}/k_{n+1},v) + (\frac{\partial \mathbf{u}^n}{\partial t}, v) +( f, v) + (\mathbf{g}^{n+1},v)
 \end{split}
$$

we rewrite the previous problem with the discrete acceleration
$$
\begin{array}{rl}
\mathbf{d}^n &=\frac{\mathbf{u}^{n+1}-\mathbf{u}^n}{k_{n+1}},\\
\mathbf{u}^{n+1} &= \mathbf{u}^n+k_{n+1} \mathbf{d}^n
\end{array}
$$
it reads
$$
\begin{split}
(\rho k_{n+1} \mathbf{d}^{n}/k_{n+1},v) + ( \mu \nabla k_{n+1} \mathbf{d}^{n}, \nabla v)  +  ( \rho k_{n+1}  \mathbf{w}^{n+1} \cdot \nabla \mathbf{d}^{n} , v) - \\
 ( p^{n+1}, \nabla \cdot v )  - (  k_{n+1} \nabla \cdot \mathbf{d}^{n}, q ) =\\
 - ( \mu \nabla u^{n}, \nabla v)  - ( \rho   \mathbf{w}^{n+1} \cdot \nabla u^{n} , v) + 
  (\frac{\partial \mathbf{u}^n}{\partial t}, v) +( f, v) + (g^{n+1},v)_{\partial \Omega} + (\nabla \cdot \mathbf{u}^n, q)
\end{split}
$$


once $$\mathbf{d}^n$$ is computed, we update $$\mathbf{u}^{n+1}$$ and $$\frac{\partial \mathbf{u}^{n+1}}{\partial t}$$ as follows

$$
\begin{array}{rl}
\mathbf{u}^{n+1} &= \mathbf{u}^n+k_{n+1} \mathbf{d}^n\\
\frac{\partial \mathbf{u}^{n+1}}{\partial t} &= 2\mathbf{d}^n - \frac{\partial u^{n}}{\partial t}
\end{array}
$$

Then using the second order Adams-Baschforth scheme
$$
\mathbf{u}^{n+1}_{AB2} = \mathbf{u}^n + \frac{k_{n+1}}{2} \left[ (1+k_{n+1}/k_n)\frac{\partial u^{n}}{\partial t} + \frac{k_{n+1}}{k_n}\frac{\partial u^{n-1}}{\partial t}\right]
$$

we compute the error between $$\mathbf{u}^{n+1}$$ and $$\mathbf{u}^{n+1}_{AB2}$$:

$$
e^{n+1} = \frac{||\mathbf{u}^{n+1}-\mathbf{u}^{n+1}_{AB2}||}{3 (1+k_{n+1}/k_n)}
$$

that is used to adjust the time step
$$
k_{n+2}=k_{n+1} \left( \frac{\varepsilon}{e^{n+1}} \right)^{1/3}
$$


Features:

 - easily adapted to convection-diffusion equations, solid mechanics (introduce velocity as an auxiliary field)
 - simple decision per time step to select or not $$k_{n+2}$$, simply reject $$k_{n+2}$$ if $$e^{n+1} > \alpha \varepsilon$$ with $$\alpha < 1$$ (in the code $$\alpha=0.7$$ and re-run current step $$n+1$$ with small time-step  
 
$$
 k_{n+1}=k_{n+1}\left( \frac{\varepsilon}{e^{n+1}} \right)^{1/3}
$$

Issues:

 1. starting this algorithm and initial time step
 2. the ringing effect which prohibits large time steps (or cancellation due to change of sign of the acceleration): averaging every $$n^*$$ iterations
 
In order to fix the ringing effect, select a frequency $$n^*$$ at which velocity and acceleration are averaged as follows:
save at $$t^*=t_n$$, $$u^*=u^n$$, and do at $$t=t_n$$
$$
t_n = t_n+\frac{k_n}{2},\quad \mathbf{u}^n=\frac{1}{2}\left(u^*+u^{n-1}\right),\quad \frac{\partial \mathbf{u}^n}{\partial t}=\frac{1}{2}\left(\frac{\partial \mathbf{u}^n}{\partial t}+\frac{\partial \mathbf{u}^{n-1}}{\partial t}\right)
$$
and at 
$$t=t_{n+1}$$
$$
t_{n+1} = t^*+\frac{k_{n+1}}{2},\quad \mathbf{u}^{n+1}=\mathbf{u}^*+\frac{1}{2}k_{n+1} \mathbf{d}^n,\quad \frac{\partial \mathbf{u}^{n+1}}{\partial t}=\mathbf{d}^n
$$


[bibliography]
.Bibliography
 - [[[kay]]] Adaptive time-stepping for incompressible flow part ii: Navier-stokes equations, DA Kay, PM Gresho, DF Griffiths, DJ Silvester - SIAM Journal on Scientific Computing, 2010 