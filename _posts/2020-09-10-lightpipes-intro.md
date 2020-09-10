# Tutorial: LightPipes for Python

**Developers of LightPipes for Python:** <br>
*Gleb Vdovin, Fred van Goor, Guyskk, Leonard Doyle, Flexible Optical B.V. (OKO Tech)*

- Package to model wave optics phenomena and beam propagation based on Scalar theory of Diffraction (Fresnel-Kirchoff)

- Versions 2.0.+, pure Python implementation using numpy, scipy and pyFFTW packages

- Works with Python 3.+ versions

- Supported on Windows, Linux and Mac 

Install as: <code>pip install LightPipes</code> <br>
[Documentation](https://opticspy.github.io/lightpipes/)

- Features: Propagation through lenses with coordinate transforms, simulation of any combination of Zernike aberrations, mode analysis in laser resonators, interferometers, inverse problems, waveguides and propagation in media with non-uniform distribution of refraction index


```python
from LightPipes import *
import numpy as np
import matplotlib
%matplotlib inline
import pylab as plt
from mpl_toolkits.mplot3d import Axes3D
from matplotlib import gridspec
import time
```


```python
print(LPversion)
LPtest()
```

    2.0.5
    Test OK
    

# Start calculations

### <code>Begin</code>
Let's begin the journey with our first LightPipes function!

All LightPipes calculations start with this built-in function which defines the field on a square grid.

<code> Field = Begin(size,wavelength,N) </code>

    Recall that 'Shift+Tab' inside a function shows the arguments it takes
    
    Seek more help with the 'help' command as help(Begin)


```python
size= 10.0*mm #size of the square grid
wavelength= 500*nm
N= 3 #grid dimension

Field= Begin(size,wavelength,N) #A square field at this point
```


```python
type(Field) #It's a LightPipes field object
```




    LightPipes.field.Field




```python
print(type(Field.field), Field.field.dtype, Field.field.shape)
```

    <class 'numpy.ndarray'> complex128 (3, 3)
    


```python
Field.field
```




    array([[1.+0.j, 1.+0.j, 1.+0.j],
           [1.+0.j, 1.+0.j, 1.+0.j],
           [1.+0.j, 1.+0.j, 1.+0.j]])



So, we've defined a uniform field with intensity 1 and phase 0 everywhere

### Apertures

- Circular aperture, <code>CircAperture </code>
- Rectangular aperture, <code>RectAperture </code>
- Gaussian aperture, <code>GaussAperture </code>

### Screens

- Circular screen, <code>CircScreen </code>
- Rectangular screen, <code>RectScreen </code>
- Gaussian screen, <code>GaussScreen </code>

Screens are the inversions of their respective apertures

#### Screens and apertures:


```python
size= 10.0*mm 
wavelength= 500*nm
N= 512 
Field= Begin(size, wavelength, N)

R= 3*mm #Aperture/Screen radius
x_shift= 2*mm; y_shift= 2*mm #Shift of the aperture/screen

Caperture= CircAperture(Field,R,x_shift,y_shift)
Iaperture= Intensity(Caperture)

Cscreen= CircScreen(Field,R,x_shift,y_shift)
Iscreen= Intensity(Cscreen)
```


```python
fig= plt.figure(figsize=(10,6))
ax1= fig.add_subplot(121)
ax2= fig.add_subplot(122)
ax1.imshow(Iaperture); ax1.axis('off'); ax1.set_title('Circular aperture')
ax2.imshow(Iscreen); ax2.axis('off'); ax2.set_title('Circular screen')
plt.show()
```


![png](output_14_0.png)


#### Combination of screens and apertures:


```python
size= 25.0*mm
wavelength= 500*nm
N= 512
Field = Begin(size, wavelength, N) 

R1= 5*mm; x_shift= 0*mm; y_shift = 0*mm; T= 0.8
F1= GaussAperture(R1,x_shift,y_shift,T,Field) #A 5mm Gaussian aperture

R2= 2*mm
F2= CircScreen(R2,x_shift,y_shift,F1) #A circular screen

R3= 4*mm
F3= CircAperture(R3,x_shift,y_shift,F2) #A circular screen

#Field intensities
I1= Intensity(F1)  
I2= Intensity(F2)
I3= Intensity(F3)
```


```python
fig= plt.figure(figsize=(16,9))
ax1= fig.add_subplot(131)
ax2= fig.add_subplot(132)
ax3= fig.add_subplot(133)
ax1.imshow(I1,cmap='hot'); ax1.axis('off'); ax1.set_title('Gaussian aperture')
ax2.imshow(I2,cmap='hot'); ax2.axis('off'); ax2.set_title('+ Circular screen')
ax3.imshow(I3,cmap='hot'); ax3.axis('off'); ax3.set_title('+ Circular aperture')
plt.show()
```


![png](output_17_0.png)



```python
t0 = time.time()
X= range(N); Y= range(N)
X, Y= np.meshgrid(X,Y) 
fig= plt.figure(figsize=(18,12))
ax= fig.gca(projection='3d')
ax.plot_surface(X, Y, I3,
                rstride=1,cstride=1,cmap='rainbow',linewidth = 0.0)
ax.set_zlabel('Intensity [a.u.]')
ax.set_title('Field intensity after the Gaussian aperture + circular screen + circular aperture')
plt.show()
print('Took', round(time.time()-t0,2), 'sec')
```


![png](output_18_0.png)


    Took 36.29 sec
    

### <code>Interpol</code>

Interpolates the field to a new grid size, grid dimension.

So with <code>Interpol</code>, we can reduce the grid dimension of the field to speed things up, especially when dealing with 3d surface plots.


```python
# Field intensity after the Gaussian aperture (using Interpol)
NewGridDimension = int(N/4)
F1_interp = Interpol(size,NewGridDimension,0,0,0,1,F1)
I1_interp = Intensity(F1_interp)
```


```python
t0 = time.time()
X= range(NewGridDimension)
Y= range(NewGridDimension)
X, Y= np.meshgrid(X,Y) 

fig = plt.figure(figsize=(18,6)) 
gs = gridspec.GridSpec(1, 2, width_ratios=[1,2]) 
ax1 = plt.subplot(gs[0])
ax2 = plt.subplot(gs[1], projection='3d')

### first subplot: cross section of the field intensity
x= []
for i in range(NewGridDimension):
    x.append((-size/2+i*size/NewGridDimension)/mm)
ax1.plot(x,I1_interp[int(NewGridDimension/2)])
ax1.set_xlabel('x [mm]')
ax1.set_ylabel('Intensity [a.u.]')

### second subplot: 3d surface plot of the field intensity
ax2.plot_surface(X,Y,I1_interp,rstride=1,cstride=1,cmap='rainbow',linewidth=0.0)
ax2.set_zlabel('Intensity [a.u.]')
ax2.set_title('Field intensity: Gaussian aperture')

plt.tight_layout()
plt.show()
print('Took', round(time.time()-t0,2), 'sec')
```


![png](output_21_0.png)


    Took 2.62 sec
    

# Free-space propagation

The [Fresnel-Kirchoff formulation](https://en.wikipedia.org/wiki/Kirchhoff%27s_diffraction_formula) is a scalar theory of diffraction and enables us to predict diffraction effects under these conditions:
- aperture is large compared to the wavelength of light, $a>>\lambda$
- field is calculated at a distance much larger than the wavelength of light, $z>>\lambda$<br/> 
    $a$: characterstic size (radius) of the diffracting aperture<br/>
    $\lambda$: wavelength<br/>
    $z$: propagation distance<br/>

Outside this regime, the predictions of this formulation is no longer accurate. Additionally, Fresnel-Kirchhoff’s formula does not correctly explain the intensity of [Poisson spot](https://physicsworld.com/a/the-spot-in-the-shadow/), a bright spot at the center observed in the near-field when light is blocked by a circular screen

The [Rayleigh-Sommerfeld formulation](https://arxiv.org/ftp/arxiv/papers/1709/1709.09727.pdf) is another scalar theory of diffraction and applies when $a >>\lambda$, typically by a factor of 10x or more. It makes a slightly different assumption about the field and its derivative at the boundaries of the aperture. As such, it overcomes the mathematical inconsistencies encountered by Fresnel-Kirchoff formulation and predicts the correct intensity of the Poisson's spot and also reproduces the diffracted field right behind the aperture ($z \gtrsim \lambda$), which Fresnel-Kirchoff fails to do. For this reason, the Rayleigh-Sommerfeld formulation is considered to be a "full" scalar solution.

Neither Fresnel-Kirchoff nor Rayleigh-Sommerfield take into account the coupled electric and magnetic vector fields, the vector nature of which cannot be ignored when $a \sim \lambda$ or $z < \lambda$


## Fresnel and Fraunhofer diffraction

- In [Fresnel-Kirchhoff](https://en.wikipedia.org/wiki/Kirchhoff%27s_diffraction_formula), the expanding wavefronts propagating away from the aperture are spherical


- [Fresnel](https://en.wikipedia.org/wiki/Fresnel_diffraction) and [Fraunhofer](https://en.wikipedia.org/wiki/Fraunhofer_diffraction) diffractions are approximations of the Fresnel-Kirchhoff formalism


- After a certain distance from the aperture and in the proximity of the optic axis, the wavefronts can be approximated as parabolic: **Fresnel or near-field approximation**


- After a sufficiently large distance and in the proximity of the optic axis, the wavefronts can be further approximated as planar wavefronts: **Fraunhofer or far-field approximation**


- Note that Fraunhofer is a special case of Fresnel, so Frensel approximation applies in the Fraunhofer regime too 

### Near and Far field:

[Fresnel number](https://en.wikipedia.org/wiki/Fresnel_number), $F =a^2/\lambda z$ coarsely determines the near/far diffraction regimes:


- When $F \geq 1 \Longrightarrow z \leq a^2/\lambda$ , this is referred to as **near-field** <br/>
Spherical wavefronts can be approximated as parabolic at this distance<br/>
Fresnel approximation applies


- When $F << 1 \Longrightarrow z >> a^2/\lambda$, the is referred to as **far-field**  <br/>
Spherical wavefronts can be approximated as planar at this distance <br/>
Fraunhofer approximations applies (Fresnel works too)

For example: With a 1 inch aperture and a 500nm beam, $a^2/\lambda$= 0.2 miles. So, the observation distance $z$ must be larger than 2 miles (an order of magnitude larger) for the Fraunhofer approximation to be valid!

### Validity ranges:
**Rayleigh-Sommerfeld**: $a >> \lambda$ and $z \gtrsim \lambda$


**Fresnel-Kirchoff**: $a >> \lambda$ and $z >> \lambda$


**Fresnel**: $z >> a >> \lambda$


**Fraunhofer**: $z >> a^2/\lambda$ 

For Fresnel and Fraunhofer: observation plane close to the optic axis such that for any observation point on the plane, $r \approx z$

*Typically a factor or 10x or more when $>>$*

### Fourier transform relations:
    
**Fresnel diffraction**: Field in the image plane is a 2D Fourier transform of the field at the aperture multiplied by a quadratic phase factor

**Fraunhofer diffraction**: Field in the image plane is a 2D Fourier transform of the field at the aperture

### Lens: a Fourier transform operator
- Field in the image plane is a 2D Fourier transform of the field at the pupil or back focal plane of the lens


- It brings to focus the diffraction pattern that would otherwise be observed at infinity (Fraunhofer diffraction pattern)

Four methods available to model free-space propagation of light:

- <code>Forvard</code> Propagate the field using FFT
- <code>Fresnel</code> Propagate the field using convolution followed by FFT
- <code>Forward</code> Propagate the field using direct integration
- <code>Steps</code> Propagate the field using a Finite difference method

### <code>Forvard(Fin,z)</code>
- Propagates the field using FFT to solve **Fresnel-Kirchoff** integral
- Simplest and fastest propagation command
- Models light propagation inside a square waveguide with reflecting walls at the grid edges
- To approximate a free space propagation, need to set the intensity near the grid edges to be negligible


```python
def pltmethod(gridsize, lamda, gridN, aperture, Intensity, zval, method, title):
    "Generate propagation plot using a single method"
    #--------------------
    # Check diffraction regime    
    FresnelNumber= round(aperture**2/(lamda*zval),4)
    print('Fresnel number (F):', FresnelNumber)
    if FresnelNumber < 0.1:
        title= title + 'Far-field diffraction (F= ' + str(FresnelNumber) + ') modeled using ' + method
    elif FresnelNumber >= 1:
        title= title + 'Near-field diffraction (F= ' + str(FresnelNumber) + ') modeled using ' + method
    else:
        title= title + 'Diffraction (F= ' + str(FresnelNumber) + ') modeled using ' + method
    #--------------------
    #Line profiles
    yy= Intensity[int(gridN/2)]
    xx=np.arange(gridN)
    xx=(xx/gridN-1/2)*gridsize/mm

    #--------------------
    fig= plt.figure(figsize=(18,8))
    fig.suptitle(title, fontsize=18)
    ax1= fig.add_subplot(121)
    ax2= fig.add_subplot(122)
    
    ax1.imshow(Intensity,cmap='hot')
    ax1.axis('off')
    ax1.set_title('After propagating '  + str(zval/m) + ' m in z using ' + method)

    ax2.plot(xx,yy)
    ax2.set_xlabel('x [mm]')
    ax2.set_ylabel('Intensity [a.u.]')
    
    plt.tight_layout(pad=7)
    plt.show()
```

#### Propagation through a circular aperture:


```python
size= 720.0*mm
wavelength= 0.5*um
N= 2304 
Field=Begin(size,wavelength,N)

title = 'Circular Aperture: '
R= 1.5*mm; x_shift= 0*mm; y_shift = 0*mm
Field_z0= CircAperture(Field,R,x_shift,y_shift)
I_z0= Intensity(Field_z0)

#Propagation distance z= [0.045m; 4.5m; 450m] ==> F= [100, 1, 0.1]
z= 450*m
   
#Propagate the field
t0= time.time()
F= Forvard(Field_z0,z)
I= Intensity(1,F)
print('Forvard: Took ', round(time.time()-t0,4), 'sec') 
```

    Forvard: Took  0.7611 sec
    


```python
print('Airy disk first minima (in Fraunhofer regime) should be at',round((1.22*z*wavelength)/(2*R)/mm, 2),'mm')
pltmethod(size, wavelength, N, R, I, z, 'Forvard', title)
```

    Airy disk first minima (in Fraunhofer regime) should be at 91.5 mm
    Fresnel number (F): 0.01
    


![png](output_34_1.png)


### <code>Fresnel(Fin,z)</code>
- Propagate the field using Convolution
- 2-5x slower compared to <code>Forvard</code> and consumes more memory 
- **Frensel diffraction** integral is converted to convolution then FFT is performed
- Unlike <code>Forvard</code>, no need to have intensities vanish at the edges, so can use a smaller grid size
- <code>Fresnel</code> results invalid if $z$ is comparable or less than the aperture at which the field is diffracted. Use <code>Forvard</code> or <code>Steps</code> that solve the full Fresnel-Kirchoff equation in these cases


```python
size= 480.0*mm
wavelength= 0.5*um
N= 1536 
Field=Begin(size,wavelength,N)

title = 'Circular Aperture: '
R= 1.5*mm; x_shift= 0*mm; y_shift = 0*mm
Field_z0= CircAperture(Field,R,x_shift,y_shift)
I_z0= Intensity(Field_z0)

#Propagation distance z= [0.045m; 4.5m; 450m] ==> F= [100, 1, 0.1]
z= 450*m
   
#Propagate the field
t0= time.time()
F1= Fresnel(Field_z0,z)
I1= Intensity(1,F1) 
print('Fresnel: Took ', round(time.time()-t0,4), 'sec')
```

    Fresnel: Took  1.6018 sec
    


```python
print('Airy disk first minima (in Fraunhofer regime) should be at',round((1.22*z*wavelength)/(2*R)/mm, 2),'mm')
pltmethod(size, wavelength, N, R, I1, z, 'Fresnel', title)
```

    Airy disk first minima (in Fraunhofer regime) should be at 91.5 mm
    Fresnel number (F): 0.01
    


![png](output_37_1.png)


### <code>Forward(Fin,z,sizenew,Nnew)</code>
- Propagates the field using direct integration of the **Fresnel-Kirchoff** integral

- Number of operations is proportional to $N^4$, where $N$ is the grid sampling

- To reduce the overall time, size of the grid can be adjusted to match the cross section of field distribution

- <code>Forward</code> also allows arbitrary sampling and size of square grid at the input and output planes


```python
size= 2.0*mm # a small grid size large enough to contain the aperture below
wavelength= 0.5*um
N= 32
Field=Begin(size,wavelength,N)

title = 'Circular Aperture: '
R= 1.5*mm; x_shift= 0*mm; y_shift = 0*mm
Field_z0= CircAperture(Field,R,x_shift,y_shift)
I_z0= Intensity(Field_z0)

#Propagation distance z= [0.045m; 4.5m; 450m] ==> F= [100, 1, 0.1]
z= 450*m

newsize= 480.0*mm
newN= 256  

#Propagate the field
t0 = time.time()
F2= Forward(Field_z0,z,newsize,newN) 
I2= Intensity(1,F2)
print('Forward: Took ', round(time.time()-t0,4), 'sec')
```

    Forward: Took  20.1499 sec
    


```python
print('Airy disk first minima (in Fraunhofer regime) should be at',round((1.22*z*wavelength)/(2*R)/mm, 2),'mm')
pltmethod(newsize, wavelength, newN, R, I2, z, 'Forward', title)
```

    Airy disk first minima (in Fraunhofer regime) should be at 91.5 mm
    Fresnel number (F): 0.01
    


![png](output_40_1.png)


#### Propagation through a Square aperture:


```python
size= 320.0*mm 
wavelength= 0.5*um
N= 1024
Field=Begin(size,wavelength,N)

title = 'Square Aperture: '
sx= 3*mm; sy= 3*mm;
x_shift= 0*mm; y_shift = 0*mm
Field_z0= RectAperture(Field,sx,sy,x_shift,y_shift)
I_z0= Intensity(Field_z0)

#Propagation distance z= [0.045m; 4.5m; 450m] ==> F= [100, 1, 0.1]
z= 450*m 

#Propagate the field
t0 = time.time()
F3=Forvard(Field_z0,z)
I3= Intensity(1,F3)
print('Forvard: Took ', round(time.time()-t0,4), 'sec')

t0 = time.time()
F4= Fresnel(Field_z0,z)
I4= Intensity(1,F4)
print('Fresnel: Took ', round(time.time()-t0,4), 'sec')
```

    Forvard: Took  0.1639 sec
    Fresnel: Took  0.6053 sec
    


```python
print('First minima (in Fraunhofer regime) should be at',round((z*wavelength)/sx/mm, 2),'mm')
#pltmethod(size, wavelength, N, sx/2, I3, z, 'Forvard', title)
pltmethod(size, wavelength, N, sx/2, I4, z, 'Fresnel', title)
```

    First minima (in Fraunhofer regime) should be at 75.0 mm
    Fresnel number (F): 0.01
    


![png](output_43_1.png)


#### Propagation through a Gaussian aperture:


```python
size= 320.0*mm 
wavelength= 0.5*um
N= 1024
Field= Begin(size,wavelength,N)

title = 'Gaussian Aperture: '
R= 1.5*mm; x_shift= 0*mm; y_shift = 0*mm; T= 0.8
Field_z0= GaussAperture(R,x_shift,y_shift,T,Field) 
I_z0= Intensity(Field_z0)

#Propagation distance z= [0.045m; 4.5m; 450m] ==> F= [100, 1, 0.1]
z= 450*m

#Propagate the field
t0 = time.time()
F6= Fresnel(Field_z0,z)
I6= Intensity(1,F6)
print('Fresnel: Took ', round(time.time()-t0,4), 'sec')
```

    Fresnel: Took  0.5901 sec
    


```python
pltmethod(size, wavelength, N, R, I6, z, 'Fresnel', title)
```

    Fresnel number (F): 0.01
    


![png](output_46_1.png)


A Gaussian beam remains Gaussian at every point as it propagates through an optical system.

#### Diffraction from a Circular Screen: [Poisson/Fresnel/Arago spot](https://physicsworld.com/a/the-spot-in-the-shadow/)

A fascinating [story on diffraction](https://physicsworld.com/a/the-spot-in-the-shadow/) that played out in the French Academy in 1818, in which a well-crafted counterargument ironically became a pillar of the opposiing view


```python
size=10.0*mm
wavelength=0.5*um
N=1024
F=Begin(size,wavelength,N)
F=GaussBeam(F,1*mm)

title = 'Circular Screen: '
R=1*mm
F=CircScreen(F,R)

z=200*mm
Farago= Fresnel(F,z)
Iarago= Intensity(1,Farago)
```


```python
pltmethod(size, wavelength, N, R, Iarago, z, 'Fresnel', title)
```

    Fresnel number (F): 10.0
    


![png](output_50_1.png)



```python
size=200.0*mm
wavelength=0.5*um
N=2000
F=Begin(size,wavelength,N)
F=GaussBeam(F,5*mm)

title = 'Circular Screen: '
R=50*mm
F=CircScreen(F,R)

z= 1600*m
Farago= Fresnel(F,z)
Iarago= Intensity(1,Farago)
```


```python
pltmethod(size, wavelength, N, R, Iarago, z, 'Fresnel', title)
```

    Fresnel number (F): 3.125
    


![png](output_52_1.png)


Note that Fresnel-Kirchhoff’s does not correctly match the measured intensity of Poisson spot. 

The [Rayleigh-Sommerfeld formulation](https://arxiv.org/ftp/arxiv/papers/1709/1709.09727.pdf) is another scalar theory of diffraction and applies when $a >>\lambda$. It makes a slightly different assumption about the field and its derivative at the boundaries of the aperture. As such, it overcomes the mathematical inconsistencies encountered by Fresnel-Kirchoff formulation and predicts the correct intensity of the Poisson's spot. 


The Rayleigh-Sommerfeld formulation also reproduces the diffracted field right behind the aperture ($z \gtrsim \lambda$), which Fresnel-Kirchoff fails to do. For this reason, the Rayleigh-Sommerfeld formulation is considered to be a "full" scalar solution. 


Experimentally though, the Fresnel-Kirchoff formulation gives more accurate results when the observation plane is many wavelengths away form the diffracting aperture. Also, the Rayleigh–Sommerfeld theory is limited to a plane surface, which limits its utility since we usually deal with curved surfaces in optics.


Lastly, neither Fresnel-Kirchoff nor Rayleigh-Sommerfield take into account the coupled electric and magnetic vector fields. The vector nature cannot be ignored, for instance when $a \sim \lambda$ or $z < \lambda$

### Using <code>Forvard</code>

- <code>Forvard</code> is a numerical solution to **Fresnel-Kirchoff** diffraction integral


- With <code>Forvard</code>, intensity at the edges of the grid must be negligible. Choose grid size >> $a$ i.e., a large grid with lots of zeros


- <code>Forvard</code> also takes a negative $z$ values, in which case it performs “back propagation” i.e., it will reconstruct the initial field from the one diffracted


### Using <code>Fresnel</code>

- <code>Fresnel</code> is a numerical solution to Fresnel diffraction integral


- <code>Fresnel</code> only takes positive $z$ values and doesn't require a large grid size


- Use <code>Fresnel</code> whenever possible to model near and far-field diffraction as long as $z>>a$ 


- When $z \sim a$, use <code>Forvard</code> instead of <code>Fresnel</code>

### Using <code>Forward</code>
- <code>Forward</code> is quite slow as computation time scales as $N^4$

- <code>Forward</code> only takes positive $z$ values

- Ideal when different grid sizes and grid elements are needed in the object and image planes

### <code>Steps(Fin,dz,nstep,refr)</code>
Finite difference method: Propagate a distance <code>nstep x dz</code> in an absorptive medium with a complex refractive index stored as the square array <code>refr</code>

- Extremely slow and results can be noisy if step sizes are large
- Use when dealing with absorptive medium (i.e., refractive index with complex components, eg., waveguides)
- <code>Steps</code> takes both positive and negative $z$ values

# Propagation through a thin lens


```python
size= 8*mm    
wavelength0= 500*nm
wavelength1= 800*nm
N= 512  

F0= Begin(size,wavelength0,N)
F0= GaussBeam(F0,w0= 2*mm) # TEM_0,0 Hermite Gauss beam

F1= Begin(size,wavelength1,N)
F1= GaussBeam(F1,w0= 2*mm) # TEM_0,0 Hermite Gauss beam

#Add a lens
f= 100*mm
F0= Lens(F0,f)
F1= Lens(F1,f)
```


```python
F= F0

#Propagate to various z-positions past the lens using Fresnel
z1= 0.33*f; Fa= Fresnel(F,z1)
z2= 0.67*f; Fb= Fresnel(F,z2)
z3= f;      Fc= Fresnel(F,z3)
z4= 1.33*f; Fd= Fresnel(F,z4)
z5= 1.67*f; Fe= Fresnel(F,z5)

plt.imshow(Intensity(1,F),cmap='rainbow')
plt.title('TEM_0,0 Hermite Gauss beam')

fig= plt.figure(figsize=(18,4))
fig.suptitle('Propagation through a f=' + str(f/mm) + 'mm lens; for ' 
             + str(round(wavelength0/nm)) + 'nm', fontsize=18)
ax1= fig.add_subplot(151); ax1.set_title(str(z1/mm) + ' mm'); ax1.imshow(Intensity(1,Fa),cmap='rainbow')
ax2= fig.add_subplot(152); ax2.set_title(str(z2/mm) + ' mm'); ax2.imshow(Intensity(1,Fb),cmap='rainbow')
ax3= fig.add_subplot(153); ax3.set_title(str(z3/mm) + ' mm'); ax3.imshow(Intensity(1,Fc),cmap='rainbow')
ax4= fig.add_subplot(154); ax4.set_title(str(z4/mm) + ' mm'); ax4.imshow(Intensity(1,Fd),cmap='rainbow')
ax5= fig.add_subplot(155); ax5.set_title(str(z5/mm) + ' mm'); ax5.imshow(Intensity(1,Fe),cmap='rainbow')
plt.tight_layout(pad=5)
plt.show()
```


![png](output_60_0.png)



![png](output_60_1.png)



```python
F= F1

#Propagate to various z-positions past the lens using Fresnel
z1= 0.33*f; Fa= Fresnel(F,z1)
z2= 0.67*f; Fb= Fresnel(F,z2)
z3= f;      Fc= Fresnel(F,z3)
z4= 1.33*f; Fd= Fresnel(F,z4)
z5= 1.67*f; Fe= Fresnel(F,z5)

fig= plt.figure(figsize=(18,4))
fig.suptitle('Propagation through a f=' + str(f/mm) + 'mm lens; for ' 
             + str(round(wavelength1/nm)) + 'nm', fontsize=18)
ax1= fig.add_subplot(151); ax1.set_title(str(z1/mm) + ' mm'); ax1.imshow(Intensity(1,Fa),cmap='rainbow')
ax2= fig.add_subplot(152); ax2.set_title(str(z2/mm) + ' mm'); ax2.imshow(Intensity(1,Fb),cmap='rainbow')
ax3= fig.add_subplot(153); ax3.set_title(str(z3/mm) + ' mm'); ax3.imshow(Intensity(1,Fc),cmap='rainbow')
ax4= fig.add_subplot(154); ax4.set_title(str(z4/mm) + ' mm'); ax4.imshow(Intensity(1,Fd),cmap='rainbow')
ax5= fig.add_subplot(155); ax5.set_title(str(z5/mm) + ' mm'); ax5.imshow(Intensity(1,Fe),cmap='rainbow')
plt.tight_layout(pad=5)
plt.show()
```


![png](output_61_0.png)


### Phase recovery using Gerchberg Saxton iteration


```python
"""
    Phase recovery from two measured intensity distributions using Gerchberg Saxton algorithm
    Adapted from PhaseRecovery.py, copyright: (c) 2017 by Fred van Goor, released under MIT license
    Modifications by Raghav K. Chhetri, September 2020
"""
wavelength= 500*nm
size= 10*mm
N= 1024
z=2*m; #propagation distance between the two fields

# Input field (near)
w0= 1.5*mm
F0= Begin(size,wavelength,N)
F0= GaussAperture(F0, w0)
I0= Intensity(0,F0)

# Output field (far)
n= 2 #number of spots
spots_dx= 1*mm
sizeforspot= spots_dx
w1= 250*um #Gaussian aperture 1/e width

spotgridN= int(N*sizeforspot/size)
Fspot= Begin(sizeforspot,wavelength,spotgridN)
Fspot= GaussAperture(Fspot,w1)

F1= Begin(size,wavelength,N)
F1= FieldArray2D(F1, Fspot, n, n, spots_dx, 3*spots_dx)
I1= Intensity(F1)
```


```python
#Plots
plt.figure(figsize=(12,6))
plt.subplot(1,2,1)
plt.imshow(I0,cmap='jet'); plt.axis('off')
plt.title('Measured Intensity (near)')
plt.subplot(1,2,2)
plt.imshow(I1,cmap='jet'); plt.axis('off')
plt.title('Measured Intensity (far)')
plt.show()
```


![png](output_64_0.png)



```python
#3d surface of field intensities
NewGridDimension = int(N/8)
F0_interp= Interpol(size,NewGridDimension,0,0,0,1,F0)
I0_interp= Intensity(F0_interp)
F1_interp= Interpol(size,NewGridDimension,0,0,0,1,F1)
I1_interp= Intensity(F1_interp)

X= range(NewGridDimension); Y= range(NewGridDimension)
X, Y= np.meshgrid(X,Y) 

fig= plt.figure(figsize=(18,7)) 
gs= gridspec.GridSpec(1, 2, width_ratios=[1,1]) 
ax1= plt.subplot(gs[0], projection='3d')
ax1.plot_surface(X,Y,I0_interp,rstride=1,cstride=1,cmap='rainbow',linewidth=0.0)
ax1.set_zlabel('Intensity [a.u.]'); ax1.set_title('Field intensity (near)')
ax2= plt.subplot(gs[1], projection='3d')
ax2.plot_surface(X,Y,I1_interp,rstride=1,cstride=1,cmap='rainbow',linewidth=0.0)
ax2.set_zlabel('Intensity [a.u.]'); ax2.set_title('Field intensity (far)')

plt.tight_layout()
plt.show()
```


![png](output_65_0.png)


Using <code>Forvard</code> for forward and back propagation 


```python
N_iterations=100 #number of iterations
N_new= N
size_new= size

#Define a field with uniform amplitude- (1) and phase (0) distribution ie = plane wave
F=Begin(size,wavelength,N)

#The iteration:
for k in range(1,N_iterations):
    #print(k)
    F=SubIntensity(I1,F) #Substitute the measured far field into the field
    F=Interpol(size_new,N_new,0,0,0,1,F);#interpolate to a new grid
    F=Forvard(-z,F) #Propagate back to the near field
    F=Interpol(size,N,0,0,0,1,F) #interpolate to the original grid
    F=SubIntensity(I0,F) #Substitute the measured near field into the field
    F=Forvard(z,F) #Propagate to the far field

#Recovered far- and near field and their phase- and intensity
#distributions; phases are unwrapped (i.e. multiples of pi removed)
F3= F #recovered far field
I3= Intensity(0,F3)
Phase_3= Phase(F3)
Phase_3= PhaseUnwrap(Phase_3)

F2= Forvard(-z,F) #recovered near field
I2= Intensity(0,F2)
Phase_2= Phase(F2)
Phase_2=PhaseUnwrap(Phase_2)

Phase_2= np.interp(Phase_2, (Phase_2.min(), Phase_2.max()), (0, 255))
Phase_3= np.interp(Phase_3, (Phase_3.min(), Phase_3.max()), (0, 255)) 
```


```python
#Plot the recovered intensity- and phase distributions
plt.figure(figsize=(18,12))

plt.subplot(2,2,1)
plt.imshow(I2,cmap='jet'); plt.axis('off')
plt.title('Recovered Intensity (near)')

plt.subplot(2,2,2)
plt.imshow(I3,cmap='jet'); plt.axis('off')
plt.title('Recovered Intensity (far)')

plt.subplot(2,2,3)
plt.imshow(Phase_2,cmap='jet'); plt.axis('off')
plt.title('Recovered phase (near)')

plt.subplot(2,2,4)
plt.imshow(Phase_3,cmap='jet'); plt.axis('off')
plt.title('Recovered phase (far)')

plt.show()
```


![png](output_68_0.png)



```python
#3d surface of field intensities
#NewGridDimension= int(N_new/8)
NewGridDimension= int(N/8)

F2_interp= Interpol(size,NewGridDimension,0,0,0,1,F2)
I2_interp= Intensity(F2_interp)
F3_interp= Interpol(size,NewGridDimension,0,0,0,1,F3)
I3_interp= Intensity(F3_interp)

X= range(NewGridDimension)
Y= range(NewGridDimension)
X, Y= np.meshgrid(X,Y) 

fig= plt.figure(figsize=(18,7)) 
gs= gridspec.GridSpec(1, 2, width_ratios=[1,1]) 
ax1= plt.subplot(gs[0], projection='3d')
ax1.plot_surface(X,Y,I2_interp,rstride=1,cstride=1,cmap='rainbow',linewidth=0.0)
ax1.set_zlabel('Intensity [a.u.]'); ax1.set_title('Recovered Field intensity (near)')
ax2= plt.subplot(gs[1], projection='3d')
ax2.plot_surface(X,Y,I3_interp,rstride=1,cstride=1,cmap='rainbow',linewidth=0.0)
ax2.set_zlabel('Intensity [a.u.]'); ax2.set_title('Recovered Field intensity (far)')

plt.tight_layout()
plt.show()
```


![png](output_69_0.png)


## Many more [examples](https://opticspy.github.io/lightpipes/examples.html) in the LightPipes documentation

# LighPipes Commands
<code>
1. Axicon(Fin, phi, n1=1.5, x_shift=0.0, y_shift=0.0)
2. BeamMix(Fin1, Fin2)
3. Begin(size, labda, N)
4. CircAperture(Fin, R, x_shift=0.0, y_shift=0.0)
5. CircScreen(Fin, R, x_shift=0.0, y_shift=0.0)
6. Convert(Fin)
7. CylindricalLens(Fin, f, x_shift=0.0, y_shift=0.0, angle=0.0)
8. FieldArray2D(Fin, Ffield, Nfieldsx, Nfieldsy, x_sep, y_sep)
9. Forvard(Fin, z)
10. Forward(Fin, z, sizenew, Nnew)
11. Fresnel(Fin, z)
12. Gain(Fin, Isat, alpha0, Lgain)
13. GaussAperture(Fin, w, x_shift=0.0, y_shift=0.0, T=1.0)
14. GaussBeam(Fin, w0, n=0, m=0, xshift=0, yshift=0, tx=0, ty=0, doughnut=False, LG=False)
15. GaussHermite(Fin, w0, m=0, n=0, A=1.0)
16. GaussLaguerre(Fin, w0, p=0, l=0, A=1.0)
17. GaussScreen(Fin, w, x_shift=0.0, y_shift=0.0, T=0.0)
18. IntAttenuator(Fin, att=0.5)
19. Intensity(Fin, flag=0)
20. Interpol(Fin, new_size, new_N, x_shift=0.0, y_shift=0.0, angle=0.0, magnif=1.0)
21. Lens(Fin, f, x_shift=0.0, y_shift=0.0)
22. LensForvard(Fin, f, z)
23. LensFresnel(Fin, f, z)
24. MultIntensity(Fin, Intens)
25. MultPhase(Fin, Phi)
26. Normal(Fin)
27. Phase(Fin, unwrap=False, units='rad', blank_eps=0)
28. PhaseSpiral(Fin, m=1)
29. PhaseUnwrap(Phi)
30. PipFFT(Fin, index=1)   
31. PlaneWave(Fin, w, tx=0.0, ty=0.0, x_shift=0.0, y_shift=0.0)
32. PointSource(Fin, x=0.0, y=0.0)
33. Power(Fin)
34. RandomIntensity(Fin, seed=123, noise=1.0)
35. RandomPhase(Fin, seed=456, maxPhase=3.141592653589793)
36. RectAperture(Fin, sx, sy, x_shift=0.0, y_shift=0.0, angle=0.0)
37. RectScreen(Fin, sx, sy, x_shift=0.0, y_shift=0.0, angle=0.0)
38. RowOfFields(Fin, Ffield, Nfields, sep, y=0.0)
39. Steps(Fin, z, nstep=1, refr=1.0, save_ram=False, use_scipy=False)
40. Strehl(Fin)   
41. SubIntensity(Fin, Intens)
42. SubPhase(Fin, Phi)
43. SuperGaussAperture(Fin, w, n=2.0, x_shift=0.0, y_shift=0.0, T=1.0)
44. Tilt(Fin, tx, ty)
45. Zernike(Fin, n, m, R, A=1.0, norm=True, units='opd')
46. ZernikeFilter(F, j_terms, R)
47. ZernikeFit(F, j_terms, R, norm=True, units='lam')
48. ZernikeName(Noll)
49. ZonePlate(Fin, N_zones, f=None, p=None, q=None, T=1.0, PassEvenZones=True)
50. noll_to_zern(j)</code>

Explore further using the built-in <code>help()</code> function

## Other interesting open-source Optics packages in Python

1. [POPPY: Physical Optics Propagation in Python](https://github.com/spacetelescope/poppy)<br>
    Simulation of Fraunhofer and Fresnel diffraction and point spread function formation, particularly in the context of astronomical telescopes
    
    
2. [BioBeam](https://maweigert.github.io/biobeam/index.html)<br>
    Multiplexed wave-optical simulations of light-sheet microscopy


3. [pyOpTools](https://pyoptools.readthedocs.io/en/latest/notebooks/basic/00-Intro.html)<br>
    Simulation of optical systems by 3D non-sequential ray tracing
