Integrations, Operators and Norms
=================================

# Integrals_Introduction Introduction
You should be able to create a mesh now. If it is not the case, get back to the section \ref Mesh.<br>

To use the tools of this sections, you have to precise the domain range using the following keywords:
<table class="manual">
<tr><th>Feel++ Keyword</th><th>Description</th></tr>
<tr><td>  elements(mesh) </td><td>All the elements of a mesh</td></tr>
<tr><td>  markedelements(mesh, id) </td><td>The precise element defined by the id.<br>It can be any element (line, surface, domain, and so on).</td></tr>
<tr><td>  faces(mesh) </td><td>All the faces of the mesh.</td></tr>
<tr><td>  markedfaces(mesh) </td><td>All the faces of the mesh which are marked.</td></tr>
<tr><td>  boundaryfaces(mesh) </td><td>All elements that own a topological dimension one below the mesh. <br>For example, if you mesh is a 2D one, `boundaryfaces(mesh)`  will return all the lines (because of dimension $$2-1=1$$).<br>These elements which have one dimension less, are corresponding to the boundary faces.</td></tr>
<tr><td>  internalelements(mesh) </td><td>All the elements of the mesh which are stricly within the domain that is to say they do not share a face with the boundary.</td></tr>
<tr><td>  boundaryelements(mesh) </td><td>All the elements of the mesh which share a face with the boundary of the mesh.</td></tr>
<tr><td>  edges(mesh) </td><td>All the edges of the mesh.</td></tr>
<tr><td>  boundaryedges(mesh) </td><td>All boundary edges of the mesh.</td></tr>
</table>




# Integral Integrals
## Integrate integrate
Thank to its finite element embedded language, Feel++ has its owned `integrate()` function.

**Interface***
```
  integrate( _range, _expr, _quad, _geomap );
```
please notice that the order of the parameter is not important, these are `boost` parameters, so you can enter them in the order you want. <br>
To make it clear, there are two required parameters and 2 optional and they of course can be entered in any order
provided you give the parameter name. If you don't provide the parameter name (that is to say `_range` = or the others) they must be entered in the order they are described
below.

Required parameters:
* `_range`  = domain of integration
* `_expr`  = integrand expression

Optional parameters:
* `_quad`  = quadrature to use instead of the default one, wich means `_Q<integer>()` where the integer is the polynomial order to integrate exactely
* `_geomap`  = type of geometric mapping to use, that is to say:
<table class="manual">
<tr><th>Feel Parameter</th><th>Description</th></tr>
<tr><td>  GEOMAP_HO </td><td>High order approximation (same of the mesh)</td></tr>
<tr><td> GEOMAP_OPT </td><td>Optimal approximation:<br> high order on boundary elements<br> order 1 in the interior</td></tr>
<tr><td> GEOMAP_01 </td><td>Order 1 approximation (same of the mesh)</td></tr>
</table>

*Example*
From `doc/manual/tutorial/dar.cpp`
```
  form1( ... ) = integrate( _range = elements( mesh ),
                            _expr = f*id( v ) );
```

From `doc/manual/tutorial/myintegrals.cpp`
```
  // compute integral f on boundary
  double intf_3 = integrate( _range = boundaryfaces( mesh ),
                             _expr = f );
```

From `doc/manual/advection/advection.cpp`
```
  form2( _test = Xh, _trial = Xh, _matrix = D ) +=
    integrate( _range = internalfaces( mesh ),
               _quad = _Q<2*Order>(),
               _expr = ( averaget( trans( beta )*idt( u ) ) * jump( id( v ) ) )
                     + penalisation*beta_abs*( trans( jumpt( trans( idt( u ) ) ) )*jump( trans( id( v ) ) ) ),
               _geomap = geomap );
```

From `doc/manual/laplacian/laplacian.cpp`
```
 auto l = form1( _test=Xh, _vector=F );
 l = integrate( _range = elements( mesh ),
                _expr=f*id( v ) ) +
     integrate( _range = markedfaces( mesh, "Neumann" ),
                _expr = nu*gradg*vf::N()*id( v ) );
```


## Integrals_Computing Computing my first Integrals
This part explains how to integrate on a mesh with Feel++ (source `doc/manual/tutorial/myintegrals.cpp` ).

Let's consider the domain $$\Omega=[0,1]^d$$ and associated meshes.<br>
Here, we want to integrate the following function
<br><center>$$
\begin{aligned}
f(x,y,z) = x^2 + y^2 + z^2
\end{aligned}
$$</center><br>
on the whole domain $$\Omega$$ and on part of the boundary $$\Omega$$.

There is the appropriate code:
```
int
main( int argc, char** argv )
{
    // Initialize Feel++ Environment
    Environment env( _argc=argc, _argv=argv,
                     _desc=feel_options(),
                     _about=about( _name="myintegrals" ,
                                   _author="Feel++ Consortium",
                                   _email="feelpp-devel@feelpp.org" ) );

    // create the mesh (specify the dimension of geometric entity)
    auto mesh = unitHypercube<3>();

    // our function to integrate
    auto f = Px()*Px() + Py()*Py() + Pz()*Pz();

    // compute integral of f (global contribution)
    double intf_1 = integrate( _range = elements( mesh ),
                               _expr = f ).evaluate()( 0,0 );

    // compute integral of f (local contribution)
    double intf_2 = integrate( _range = elements( mesh ),
                               _expr = f ).evaluate(false)( 0,0 );

    // compute integral f on boundary
    double intf_3 = integrate( _range = boundaryfaces( mesh ),
                               _expr = f ).evaluate()( 0,0 );

    std::cout << "int global ; local ; boundary" << std::endl
              << intf_1 << ";" << intf_2 << ";" << intf_3 << std::endl;
}
```




# Operators Operators
## Project project
It is also possible to make projections with the library.

**Interface***
```
  project( _range, _space, _expr, _geomap );
```

Required parameters:
* `_space`: the space in which lives the projected expression, it should be a nodal function space
* `_expr`: the expression to project

Optional parameters:
* `_range`: the domain for the projection. Default = all elements from `space->mesh()`
* `_geomap`: type of geometric mapping. Default = `GEOMAP_OPT`

*Example*
From `doc/manual/laplacian/laplacian.cpp`
```
  element_type e( Xh, "e" );
  e = project( _space = Xh,
               _range = elements( mesh ),
               _expr = g );
```

From `doc/manual/heatns/convection_run.cpp`
```
tn = project( _space = Xh->functionSpace<2>(),
              _range = elements( mesh ),
              _expr = constant( 300 ) );
```



## Mean mean
Let $$f$$ a bounded function on domain $$\Omega$$. You can evaluate the mean value:
<br><center>$$
  \begin{aligned}
 \bar{f}&=\frac{1}{|\Omega|}\int_\Omega f\\
&=\frac{1}{\int*mits_\Omega 1}\int_\Omega f.
  \end{aligned}
$$</center><br>

**Interface***
```
  mean( _range, _expr, _quad, _geomap );
```

Required parameters:
* `_range` = domain of integration
* `_expr` = mesurable function

Optional parameters:
* `_quad` = quadrature to use. Default = \lstinline!_Q<integer>()!
* `_geomap` = type of geometric mapping. Default = `GEOMAP_OPT`

*Example*
From `doc/manual/stokes/stokes.cpp`
!CODEFILE "code/stokes.cpp" mean



# Norms Norms
## NormL2 normL2
Let $$f \in L^2(\Omega)$$ you can evaluate the L2 norm:
<br><center>$$
  \begin{aligned}
\parallel f\parallel_{L^2(\Omega)}=\sqrt{\int_\Omega |f|^2}
  \end{aligned}
$$</center><br>

**Interface***
```
  normL2( _range, _expr, _quad, _geomap );
```
or squared norm:
```
  normL2Squared( _range, _expr, _quad, _geomap );
```

Required parameters:
* `_range` = domain of integration
* `_expr`  = mesurable function

Optional parameters:
* `_quad`  = quadrature to use. Default = `_Q<integer>()`
* `_geomap`  = type of geometric mapping. Default = `GEOMAP_OPT`

*Example*
From `doc/manual/laplacian/laplacian.cpp`
```
  double L2error =normL2( _range=elements( mesh ),
                          _expr=( idv( u )-g ) );
```

From `doc/manual/stokes/stokes.cpp`
!CODEFILE "code/stokes.cpp" norml2

##  normH1

In the same idea, you can evaluate the H1 norm or semi norm, for any
function $$f \in H^1(\Omega)$$:

$$\begin{aligned}
 \parallel f \parallel_{H^1(\Omega)}&=\sqrt{\int_\Omega |f|^2+|\nabla f|^2}\\
&=\sqrt{\int_\Omega |f|^2+\nabla f * \nabla f^T}\\
|f|_{H^1(\Omega)}&=\sqrt{\int_\Omega |\nabla f|^2}
\end{aligned}$$

where $$*$$ is the scalar product $$\cdot$$ when $$f$$ is a scalar
field and the frobenius scalar product $$:$$ when $$f$$ is a vector
field

**Interface***
```
  normH1( _range, _expr, _grad_expr, _quad, _geomap );
```
or semi norm:
```
  normSemiH1( _range, _grad_expr, _quad, _geomap );
```

Required parameters:
* `_range` = domain of integration
* `_expr` = mesurable function
* `_grad_expr` = gradient of function (Row vector!)

Optional parameters:
* `_quad` = quadrature to use. Default = `_Q<integer>()`
* `_geomap` = type of geometric mapping. Default = `GEOMAP_OPT`


*Example*
With expression:
```cpp
  auto g = sin(2*pi*Px())*cos(2*pi*Py());
  auto gradg = 2*pi*cos(2* pi*Px())*cos(2*pi*Py())*oneX() \
	           -2*pi*sin(2*pi*Px())*sin(2*pi*Py())*oneY();
// There gradg is a column vector!
// Use trans() to get a row vector
  double normH1_g = normH1( _range=elements(mesh),\
  			                _expr=g,\
			                _grad_expr=trans(gradg) );
```
With test or trial function `u`
```cpp
  double errorH1 = normH1( _range=elements(mesh),\
  			               _expr=(u-g),\
    			           _grad_expr=(gradv(u)-trans(gradg)) );
```



## normLinf

Let $$f$$ a bounded function on domain $$\Omega$$. You can evaluate the infinity norm:

$$\parallel f \parallel_\infty=\sup_\Omega(|f|)$$

**Interface**
```cpp
  normLinf( _range, _expr, _pset, _geomap );
```

Required parameters:
* `_range` = domain of integration
* `_expr` = mesurable function
* `_pset` = set of points (e.g. quadrature points)

Optional parameters:
* `_geomap` = type of geometric mapping. Default = `GEOMAP_OPT`


*Example*:
```cpp
  auto uMax = normLinf( _range=elements(mesh),\
		        _expr=idv(u),\
		        _pset=_Q<5>() );
```
