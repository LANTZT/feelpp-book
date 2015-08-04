Defining a Model {#TutorialModel}
================



# Introduction {#Intro}

It is well known an equation can describe a huge range of physical
problems.  Each of theses problems will have a particular environment,
but the equation to solve will be the same.  To make our program
applicable to theses range of problem, we have defined a model.

# What is a model {#What}

A model is defined by :
- a Name
- a Description
- a Model
- [Parameters](#Parameters)
- [Materials](#Materials)
- [Boundary Conditions](#BoundaryConditions)
- [Post Processing](#PostPro)

## Parameters {#Parameters}
A parameter is a non physical property for a model.

## Materials {#Materials}
To retrieve the materials properties, we use :   
```  ModelMaterials materials = model.materials(); ```  

And to apply them :   

!CODEFILE "code/model1.cpp"   


## BoundaryConditions {#BoundaryConditions}
Thanks to GiNaC, we handle boundary conditions (Dirichlet, Neumann, Robin) as expression.
You have to indicate in the json file the quantity to handle (velocity, pressure...) and the associated expression.   

```map_scalar_field<2> bc_u { model.boundaryConditions().getScalarFields<2>("heat","dirichlet") };```  


We can apply theses boundary condition this way
```
  for(auto it : bc_u){
    if(boption("myVerbose") && Environment::isMasterRank() )
      std::cout << "[BC] - Applying " << it.second << " on " << it.first << std::endl;
    a+=on(_range=markedfaces(mesh,it.first), _rhs=l, _element=u, _expr=it.second );
  }```



## PostProcessing {#PostPro}

# Example {#Example}
We have set up an example : an anisotropic laplacian.   
!CODEFILE "code/model2.cpp"


