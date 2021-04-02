
Adaptive Step-size Euler Method - A Visualisation




The dynamical system studied in the following example will be of the form:

 <p align="center">
 <img src="https://render.githubusercontent.com/render/math?math=\mathbf{x}=\begin{bmatrix}
x_1\\
x_2
\end{bmatrix}">
   <img src="https://render.githubusercontent.com/render/math?math=\mathbf{\dot{x}}=\begin{bmatrix}
-16x_1 %2B 12x_2 %2B 16\text{cos}(t)-13\text{sin}(t) \\
12x_1-9x_2-11\text{cos}(t) %2B 9\text{sin}(t) 
\end{bmatrix}">
 </p>
 Simulated over 3 time intervals, with an initial condition of  <img src="https://render.githubusercontent.com/render/math?math=\begin{bmatrix}
1 \\
0
\end{bmatrix}">.

Adaptive integration techniques are based on the idea of performing a full Euler step alongside a 2nd-order step such as Heun's method. The difference between these two states is then taken as an error, if this error drops the stepsize is increased, if it rises then the stepsize is decreased. 

The Euler prediction of the next state is defined as:
 <p align="center">
   <img src="https://render.githubusercontent.com/render/math?math=s_{i %2B 1}^e=s_i %2B hf(s_i)">
 </p>
With the 2nd-order Heun prediction defined as:
 <p align="center">
   <img src="https://render.githubusercontent.com/render/math?math=s_{i %2B 1}^h=s_i %2B \frac{1}{2} h(f(s_i) %2B f(s_{i %2B 1}^e))">
 </p>
 Thus resulting in an overall error of:
  <p align="center">
   <img src="https://render.githubusercontent.com/render/math?math=e_{i %2B 1} \approx ||s_{i %2B 1}^e-s_{i %2B 1}^h|| =  \frac{1}{2} ||f(s_i)-f(s_{i %2B 1}^e)||">
 </p>
 

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/assets/htmlplots/adaptive_euler.html" height="600" width="100%"></iframe>

Now lets try a slightly trickier example, the start up of a non-isothermal semi-batch reactor. The states are:
<p align="center">
   <img src="https://render.githubusercontent.com/render/math?math=\mathbf{x}=\begin{bmatrix}
C_a \\
C_b \\
C_C \\
T \\
V \\
\end{bmatrix}">
 </p>

 With the dynamics found in [Fogler (2020)](http://umich.edu/~elements/5e/).
<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/assets/htmlplots/adaptive_euler_reactor.html" height="600" width="100%"></iframe>

It can be seen that due to the stiffness of the differential equation particularly at the start and middle of the simulation, the adaptive method reduces the step size automatically due to errors between the first and second order prediction. 

