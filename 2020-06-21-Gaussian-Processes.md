---
title:  "Gaussian Processes - A brief tutorial"
layout: single
author_profile: true 
excerpt: "A small Gaussian Process tutorial adapted from a course I recently gave to incoming Masters students to Dongda Zhang's group at the University of Manchester. I've also presented a similar talk at Seoul National University"
header:
  overlay_image: /assets/images/output_15_4.png
  overlay_filter: rgba(100,100,100, 0.5)
  caption: "Gaussian process prior"
classes: wide
---

### Begin by importing python modules:

```python
import numpy as np 
import matplotlib.pyplot as plt 
import seaborn as sb
```

### Single-variable Gaussian Distributions

First we'll demonstrate how to draw samples from a Gaussian distribution in Python (also known as the normal distribution).
Within numpy there is a `numpy.random` package that is used to generate random samples. 
We can specify a mean and variance of our normal distribution as well as the number of samples, and it will return a vector of samples from this distribution.


```python
# taking 1000 samples from a Gaussian distribution with mean 0 and standard deviation 1 

sample = np.random.normal(0,1,1000) 

# plotting the samples as a histogram 
sb.distplot(sample)
plt.grid()
plt.xlabel('$x$')
plt.ylabel('$P(x)$')
```

<p align="center">
<img src="/assets/images/output_4_1.png" width="50%">
</p>

### Mutivariate Gaussian distributions in Python

Making the extension to sampling from multivariate Gaussian distributions, as opposed to specifying scalar values for our mean and variance, we provide a vector and matrix value for our mean and covaraiance respectively. Other than that the `multivariate_normal` function works just as before, specifying a number of samples (in this case 1000) returns a vector of samples generated from this multivariate Gaussian distribution. 

Plotting a multivariate distribution specified by mean <img src="https://render.githubusercontent.com/render/math?math=[0,0]"> and covariance matrix <img src="https://render.githubusercontent.com/render/math?math=\\\begin{bmatrix}0.5 & 0.5\\0.5 & 1\\ \end{bmatrix}">

This mean vector states the mean of the variables <img src="https://render.githubusercontent.com/render/math?math=x_1"> and <img src="https://render.githubusercontent.com/render/math?math=x_2">  are both 0, the variance of <img src="https://render.githubusercontent.com/render/math?math=x_1"> is 0.5, the variance of <img src="https://render.githubusercontent.com/render/math?math=x_2"> is 1, the covariance between <img src="https://render.githubusercontent.com/render/math?math=x_1"> and <img src="https://render.githubusercontent.com/render/math?math=x_2"> is 0.5. 

We first ask Python to generate 1000 samples from this distribution, then plot the distribution of this data using the seaborn library. 


```python
mu     = np.array([0,0])                                      # 2 dimensional mean vector 
cov    = np.array([[0.5,0.5],[0.5,1]])                      # 2x2 covariance matrix 
# taking 1000 samples from this 2D Gaussian 
sample = np.random.multivariate_normal(mu,cov,100)            
print('mean vector: ',mu)
print('covariance matrix: ','\n', cov)
# plotting this 2 dimensional Gaussian distribution 
# using the seaborn library                      
sb.kdeplot(sample[:,0],sample[:,1],shade=True,cbar=True)
plt.xlabel('$x_1$')
plt.ylabel('$x_2$')
```
<p align="center">
<img src="/assets/images/output_6_2.png" width="50%">
</p>

### Visualising a sample from a multivariate Gaussian distribution. 

Taking our previous 2-dimensional Gaussian we take a sample using the `multivariate_normal` function.
Sampling within this distribution will give us a value for <img src="https://render.githubusercontent.com/render/math?math=x_1"> and <img src="https://render.githubusercontent.com/render/math?math=x_2">.

A way of visualising this sample which will become extremely useful later (and is the basis of Gaussian processes) is plotting the integer dimensions on the x-axis, and the respective sampled dimensional on the y-axis. 


```python

# taking a sample from this 2-D gaussian distribution #
sample_point = np.random.multivariate_normal(mu,cov)  

#---------------- visualising the sample -------------# 

fig, (ax1, ax2) = plt.subplots(ncols=2,figsize=(10,3)) 
fig.subplots_adjust(wspace=0.35)
sb.kdeplot(sample[:,0],sample[:,1],shade=True,ax=ax1)
plt.xlabel('$x_1$'); plt.ylabel('$x_2$')
ax1.scatter(sample_point[0],sample_point[1],c='k',marker='o')
ax1.axis([-2,2,-2,2])
ax1.set_xlabel('$x_1$'); ax1.set_ylabel('$x_2$')
ax1.plot([sample_point[0],sample_point[0]],[-10,sample_point[1]],c='k',linestyle='--')
ax1.plot([-10,sample_point[0]],[sample_point[1],sample_point[1]],c='k',linestyle='--')
ax2.plot(np.arange(1,len(sample_point)+1),sample_point)
ax2.scatter(np.arange(1,len(sample_point)+1),sample_point,c='k',marker='o')
ax2.set(xlabel='Gaussian Sample Dimension #', ylabel='Value of sample in each dimension')
ax2.set_xticks([1,2])
plt.show()
```


<p align="center">
<img src="/assets/images/output_10_0.png" width="100%">
</p>

Now what if we sample from a Gaussian distribution with more than two variables? We can no longer plot the distribution itself but we can continue plotting as we have done above. I will also plot the covariance matrix that has been used to generate these samples.



<p align="center">
<img src="/assets/images/output_15_0.png" width="100%">
</p>



<p align="center">
<img src="/assets/images/output_15_1.png" width="100%">
</p>




<p align="center">
<img src="/assets/images/output_15_2.png" width="100%">
</p>




<p align="center">
<img src="/assets/images/output_15_3.png" width="100%">
</p>




<p align="center">
<img src="/assets/images/output_15_4.png" width="100%">
</p>


Notice that sampling from a Gaussian distribution, with this specific covariance matrix, results in smooth function like curves! 

But why this covariance matrix? 

Well if the covariance matrix was simply the identity, then it would imply that neighbouring indices on the x axis are not correlated, resulting in something like this...

<p align="center">
<img src="/assets/images/identity_cov.png" width="100%">
</p>
 
 By specifying the off-diagonal elements in this specific way, we include knowledge as to how neighbouring indices are related to each other, resulting in smooth curves! 
 
 But where does this covariance matrix come from? 

### Generating a covariance matrix for higher dimension Gaussian distributions

Of course we don't wish to keep having to type in a covariance matrix every single time we want to draw from a Gaussian distribution.
Taking a slight detour lets think about how best to generate a covariance matrix, this is important because it's how we incorporate prior knowledge into our function approximation. 

You have a stirred tank reactor and you wish to understand the distribution of temperature within the vessel. You obviously can't take a measurement of all temperatures at once, however you can take the temperature at certain coordinates within the reactor and then 'fill in the gaps'. The question is, how does the temperature vary _between_ measured locations. 

Well intuitively you would say that the temperature of the area surrounding a particular measurement will definitely be close to the temperature at the measurement location. What about the temperature slightly further away? Well you could say that it's going to be similar to the temperature at the measurement but it's relation to the temperature at the measured location definitely decreases the further you move away from the measurement. 

So this seems to be a good assumption to create a covariance function, the assumption that the correlation between a measured location within the reactor and some other location decreases the further away you are. 

Now heading back to our finite dimension Gaussian distribution for the time being, maybe if we're plotting the sample as above (variables on the x-axis and values on the y) we could say that as the distance between Gaussian indices (on our x-axis) increases then our covariance decreases. 

The covariance between 2 independent variables is 0. 



Now hold on... we're just taking large samples at this point, how do we generate functions? Well this is what turns a Gaussian distribution into a Gaussian _process_. In Gaussian processes, _each coordinate_ along our independent variable (e.g. location without our reactor, or time for example) is modelled as an independent variable of a Gaussian distribution. 
So to model a continuous function over a single independent variable x (for example time) we need an infinite dimension Gaussian distribution!? This is a tricky relation and at the core of Gaussian processes.

We can query this infinite dimension gaussian distribution (and therefore take a sample at any point along our x-axis) by making the following replacement. 

Instead of specifying our distribution with a mean **vector** and covariance **matrix**, we replace these with a mean **function** and covariance **function** (also known as a kernel). The intuition here is that a function can just be seen as a vector of infinite length, and vice versa. This functional form of the mean and covariance allows us to query 'between' the discrete indices that are described by a mean vector or covariance matrix. 

 <p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\mu = [\mu_1,\mu_2,\mu_3,...,\mu_n] \rightarrow \mu(x)"> 
</p>


Now our Gaussian Process (infinitely dimensioned Gaussian distribution) is fully specified by a mean **function** and a covariance **function**.

So Generally for a function: 

 <p align="center">
<img src="https://render.githubusercontent.com/render/math?math=y = f(X) %2B \epsilon"> 
</p>

a Gaussian process specifies f as 

 <p align="center">
<img src="https://render.githubusercontent.com/render/math?math=f \sim \mathcal{GP}(\mu(x),k(x,x^*))"> 
</p>


Where <img src="https://render.githubusercontent.com/render/math?math=x^*">  is the point we wish to evaluate our gaussian process at. Most of the time we just say that the mean function is 0 to avoid complication although it can often be a linear or quadratic function of the input data <img src="https://render.githubusercontent.com/render/math?math=x">. This assumption is perfectly fine when we normalise our data around the origin.

 <p align="center">
<img src="https://render.githubusercontent.com/render/math?math=f \sim \mathcal{GP}(0,k(x,x^*))"> 
</p>

### What function to use?
Shown here is the squared exponential kernel function, which was used to create the covariance matrices in the above plots. It relates the distance between samples to how correlated they are (off-diagonal elements in the covariance matrix).

<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\text{squared_exp}(\text{l},\sigma,x_1,x_2,\sigma_n) = \sigma^2 \cdot exp \left[-(\frac{x_1-x_2}{2l^2})^2\right] + \sigma_n"> 
</p>


Parameters l and <img src="https://render.githubusercontent.com/render/math?math=\sigma"> come into play later.
When x has more than one input variable, a norm can be used, meaning that the covariance function always returns a single number and never a vector. This allows for the extension to a number of input variables.

Reminder: a norm is any function that roughly relates to a distance between two points. e.g.
 
 <p align="center">
<img src="https://render.githubusercontent.com/render/math?math=||x||_2 = \sqrt{\sum _{i=1} x_i\cdot x_i }"> 
</p>


i.e. 

 <p align="center">
<img src="https://render.githubusercontent.com/render/math?math=k(\mathbf{x},\mathbf{x_*}): \mathbb{R}^n, \mathbb{R}^n \rightarrow \mathbb{R}"> 
</p>

```python
def squared_exp(params,x1,x2):
    # Defining a covariance function 
    l = params[0]     # length scale
    sig = params[1]   # variance
    sig_n = params[2] # noise parameter 
    return (sig**2 * np.exp(-((x1-x2)/2*l**2)**2))+sig_n

#--- plotting how covariance varies between two points ---#
x_1 = 0 
x_2 = np.linspace(-4,4,100)
x_2_r = np.linspace(-4,4,9)
plt.figure()
for i in np.linspace(0.5,2,5):
  cov_fun_r = squared_exp([i,1,0.1],x_1,x_2_r)
  cov_fun = squared_exp([i,1,0.1],x_1,x_2)
  plt.plot(x_2,cov_fun,linewidth='4',alpha=0.2,label=str(np.round(i,3))); plt.grid()
  plt.scatter(x_2_r,cov_fun_r,linewidth='4');
plt.xlabel('Distance from $x_1$'); plt.ylabel('Covariance')
plt.legend()

plt.show()
```


<p align="center">
<img src="/assets/images/output_12_0.png" width="50%">
</p>


We can now take every point in our dataset, and produce it's covariance value against every other point in the dataset. This will hold the form of a matrix, other words the covariance matrix. 
This matrix is symmetrical as the distance from A -> B is obviously equal to the distance from B -> A. 


```python
# defining a function that takes a set of datapoints and a covariance function
# and returns a covariance matrix

def cov(params,kernel,x,x_star):
    if len(x) == len(x_star):                # if the data is square...
        cov_m = np.zeros((len(x),len(x_star)))
        for i in range(len(x)):
            for j in range(i+1,len(x_star)): # only need to compute upper triangular
                cov_m[i,j] = kernel(params,x[i],x_star[j])
        cov_m += cov_m.T                     # append the transpose to gain square matrix
        for i in range(len(x)):
            for j in range(len(x)):          # make sure diagonal is only added once 
                cov_m[i,j] = kernel(params,x[i],x_star[j])
        return cov_m  
    
    else:                                    # otherwise just produce full rectangular matrix 
        cov_m = np.zeros((len(x),len(x_star)))
        for i in range(len(x)):
            for j in range(len(x_star)):
                cov_m[i,j] = kernel(params,x[i],x_star[j])
    return cov_m 

x_test = np.arange(10)
cov_test = cov([0.5,1,0.1],squared_exp,x_test,x_test)
plt.title('covariance matrix')
plt.imshow(cov_test)
plt.colorbar()
```
<p align="center">
<img src="/assets/images/output_14_1.png" width="50%">
</p>


Now we have a functional form of the covariance we can use this to create covariance matrices of any size. Here is the code used to generate the above plots.

```python
# plotting samples from increasingly dimensional Gaussian distributions using 
# a smooth squared exponential covariance function 

for i in np.logspace(0.4,2,5):
    gauss_dimensions = int(i)
    x = np.linspace(0,20,gauss_dimensions)
    l = 1
    sig = 1
    sig_n = 0.1
    params = [l,sig,sig_n]
    sample_num = 10
    mu = [0 for i in range(len(x))]
    cov_x = cov(params,squared_exp,x,x)
    #cov_x = np.eye(len(x))
    fig, (ax1, ax2) = plt.subplots(ncols=2,figsize=(10,3))
    im = ax1.imshow(cov_x)
    plt.colorbar(im, ax=ax1)
    ax1.title.set_text('Covariance Matrix')
    ax2.set_xlabel('$x$'); ax2.set_ylabel('$f(x)$')
    ax2.plot(np.arange(gauss_dimensions),mu,label='mean',c='r',linestyle='--')
    plt.fill_between(np.arange(gauss_dimensions), mu+np.diagonal(cov_x)*2, mu-np.diagonal(cov_x)*2,label='95% variance',alpha=0.2)
    for j in range(sample_num):
        sample = np.random.multivariate_normal(mu,cov_x)
        ax2.plot(np.arange(len(sample)),sample,label='sample '+str(j+1))
    ax2.legend(loc='center left', bbox_to_anchor=(1, 0.5))
    plt.title('Gaussian distribution dimensions: '+str(int(i)))
    plt.show()
```


<p align="center">
<img src="/assets/images/output_15_0.png" width="100%">
</p>



<p align="center">
<img src="/assets/images/output_15_1.png" width="100%">
</p>




<p align="center">
<img src="/assets/images/output_15_2.png" width="100%">
</p>




<p align="center">
<img src="/assets/images/output_15_3.png" width="100%">
</p>




<p align="center">
<img src="/assets/images/output_15_4.png" width="100%">
</p>


## Inclusion of data (conditioning)

Now we have this infinite dimension Gaussian distribution (AKA a Gaussian process), which we can query in any dimension we want. 

In short, by specifying the what the function value is along certain 'dimensions' we can observe the resulting still-infinite distribution, with a fixed value at a specific location.

The conditioning a Gaussian has the form of another Gaussian as seen in the following image. By conditioning we effectively 'set' the gaussian distribution to a particular dimension and infer the distribution of the resulting Gaussian.

<p align="center">
<img src="/assets/images/output_8_1.png" width="50%">
</p>

This conditioned Gaussian is shown as follows:

 <p align="center">
<img src="https://render.githubusercontent.com/render/math?math=f_* | X_*,X,f \sim N(K(X_*,X)K(X,X)^{-1}f,K(X_*,X_*)-K(X_*,X)K(X,X)^{-1}K(X,X_*)) "> 
</p>


But it can effectively can be thought as choosing where the orange distribution below is specified (up/down) and the implied distribution is where out Gaussian process is drawn from. 

### Important Note
Notice the inverse of the covariance matrix of the data <img src="https://render.githubusercontent.com/render/math?math=K(X,X)^{-1}"> , computationally this is done in <img src="https://render.githubusercontent.com/render/math?math=O(n^3)"> time, so if you have 10 times as many datapoints, the matrix takes 1000 times longer to invert. This is the main bottle-neck of Gaussian processes and therefore they are often restricted to low-datasizes. 



```python
# our data is 5 datapoint, and our 'test' is 100 data points which we'll plot in python as a smooth curve. 
# it looks smooth in Python using 'plot' however it really is just a vector of length 100 with the lines connected.


x_data = np.linspace(-2.5,12.5,7)
y_data = np.sin(x_data)
x_test = np.linspace(-5,15,100)
x_real = np.linspace(-5,15,100)
y_real = np.sin(x_real)

kernel = squared_exp
params = [1,1,0.1]

k_ss      = cov(params,kernel,x_test,x_test) # covariance matrix of evaluation points
k_sl      = cov(params,kernel,x_data,x_test) # rectangular covariance matrix of evaluation and data points 
k_sf      = k_sl.T                           # making the rectangle 'lie down'
k_og      = cov(params,kernel,x_data,x_data) # covariance matrix of original data points
k_inv     = np.linalg.inv(k_og)              # inverting original covariance matrix for inference 

post_mean = k_sf @ k_inv@y_data              # posterior mean 
post_cov  = k_ss-k_sf@k_inv@k_sl             # posterior augmented covariance matrix     

plt.figure()
plt.subplot(1, 2, 1)
plt.title('original precision matrix')
plt.imshow(k_inv)
plt.subplot(1, 2, 2)
plt.title('conditioned precision matrix')
plt.imshow(post_cov)
plt.show()
```


<p align="center">
<img src="/assets/images/output_19_0.png" width="75%">
</p>




```python
# --- taking samples from posterier gaussian distribution --- # 

sample_num = 0 # number of samples
fig, (ax1, ax2) = plt.subplots(ncols=2,figsize=(10,3))
pos = ax1.imshow(post_cov)
fig.colorbar(pos,ax=ax1)
ax1.title.set_text('Covariance Matrix')
ax2.plot(x_real,y_real,c='k',label='real function')
ax2.plot(x_test,post_mean,label='mean',c='r',linestyle='--')
plt.fill_between(x_test, post_mean+np.diagonal(post_cov)*2, post_mean-np.diagonal(post_cov)*2,label='95% variance',alpha=0.2)
plt.title('Gaussian process conditioned on test data')
ax2.scatter(x_data,y_data,c='b')
ax2.legend(loc='center left', bbox_to_anchor=(1, 0.5))
ax2.grid()
ax2.set_xlabel('$x$'); ax2.set_ylabel('$f(x)$')
plt.show()

# plotting with some samples from the conditional distribution 

sample_num = 10 # number of samples
fig, (ax1, ax2) = plt.subplots(ncols=2,figsize=(10,3))
pos = ax1.imshow(post_cov)
fig.colorbar(pos,ax=ax1)
ax1.title.set_text('Covariance Matrix')
ax2.plot(x_real,y_real,c='k',label='real function')
ax2.plot(x_test,post_mean,label='mean',c='r',linestyle='--')
plt.fill_between(x_test, post_mean+np.diagonal(post_cov)*2, post_mean-np.diagonal(post_cov)*2,label='95% variance',alpha=0.2)
for j in range(sample_num):
    sample = np.random.multivariate_normal(post_mean,post_cov)
    ax2.plot(x_test,sample)
plt.title('Gaussian process conditioned on test data')
ax2.scatter(x_data,y_data,c='b')
ax2.legend(loc='center left', bbox_to_anchor=(1, 0.5))
ax2.grid()
ax2.set_xlabel('$x$'); ax2.set_ylabel('$f(x)$')
plt.show()
```


<p align="center">
<img src="/assets/images/output_20_0.png" width="100%">
</p>




<p align="center">
<img src="/assets/images/output_20_1.png" width="100%">
</p>


As a nice example, here I'll show the prior GP, and then the posterior after adding a single data-point each time. You can see how the resulting distribution-over-functions changes when data is provided. Effectively eliminating the probability of drawing a sample that doesn't go through the data point. 

<p align="center">
<img src="/assets/images/download.png" width="100%">
</p>
<p align="center">
<img src="/assets/images/download-1.png" width="100%">
</p>
<p align="center">
<img src="/assets/images/download-2.png" width="100%">
</p>
<p align="center">
<img src="/assets/images/download-3.png" width="100%">
</p>
<p align="center">
<img src="/assets/images/download-4.png" width="100%">
</p>
<p align="center">
<img src="/assets/images/download-5.png" width="100%">
</p>
<p align="center">
<img src="/assets/images/download-6.png" width="100%">
</p>
<p align="center">
<img src="/assets/images/download-7.png" width="100%">
</p>