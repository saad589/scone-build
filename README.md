# SCONE build guide for dummies 

SCONE is a Monte Carlo radiation transport modeling/development tool. 
It is being developed and maintained by the nuclear energy research group of Cambridge University. 
SCONE is open source and the public repository is hosted [here](https://bitbucket.org/Mikolaj_Adam_Kowalski/scone/src/develop/). 

The official installation guide can be found [here](https://scone.readthedocs.io/en/latest/Installation.html). 
This "for dummies" guide is intended to make the build process a bit easier for novice users/developers. 

This guide contains:

1. Things that you need as prerequisites. 
2. How to download/clone the source code from the publicly hosted repository.
3. Configure the build process. 
4. Compile the source code into an executable binary. 
6. Create a symbolic link.
7. Download the ACE-formatted nuclear cross-section library.
8. Configure a directory file for the downloaded library to be used with SCONE.
8. Write a sample input file and run it.

## Prerequisites

### OS
This guide is specifically prepared for Ubuntu/WSL. Ubuntu based distros (e.g., Lubuntu, Pop OS) should work as well. 

If you have a dedicated Ubuntu machine (or VM/Docker), that's great!

If you are on a Windows machine, Windows Subsystem for Linux or WSL might just be the best option for you. It is a compatibility layer that lets you run a complete Ubuntu terminal environment (or any Linux binary executables) natively on Windows. Go to Microsoft Store and write "Ubuntu" on the search bar. Installation should be trivial. Ensure that the "Virtual Machine Platform" and "Windows Subsystem for Linux" features are selected in "Windows Features". SCONE runs on WSL perfectly fine. 

### Required repositories
Before attempting SCONE compilation, the following software packages should be present on the machine, 

1. A meta-package named “build-essential” that includes the GNU compiler collection, GNU debugger, and other development libraries and tools required for compiling software. 
2. Fortran (must be version 7)
3. CMake (must be version 3.10+)

Additionally,

4. Git
5. OpenMP
6. LAPACK and BLAS
7. pFUnit (optional)

Open a terminal (or in Windows, a Ubuntu terminal environment) and issue the following commands,

```
$ sudo apt update
$ sudo apt install build-essential
$ sudo apt install gfortran
$ sudo apt install cmake
```
When you try to install gfortran with apt, it installs the latest available version (e.g., 9.x.x or 10.x.x). You can check the version with ``gfortran --version``. However SCONE requires gfortran version 7.x.x, otherwise, it will not compile! 

So you need to downgrade it. 

> It is worthwhile to cover some basics here so you can appreciate what is happening next. When you type in ``gfortran``, it is actually a symbolic link located at ``/usr/bin/gfortran``. This location (along with other locations) is stored in an environment variable called ``PATH``. To view your ``PATH`` variable, type ``echo $PATH``. To view all the environment variables for the current user, type ``env``.    

>Right now ``/usr/bin/gfortran`` points to, for example, 
``/usr/bin/x86_64-linux-gnu-gfortran-9`` which is the 9.x.x binary.
Whereas you need to make ``/usr/bin/gfortran`` to point to a 7.x.x
binary.

The easiest way to install gfortran 7.x.x is, obviously, using the apt utility. If you do a repository search using ``apt-cache search gfortran``, there is gfortran-7 on the list! Install this repository by typing, 

```
$ sudo apt isntall gfortran-7
```
Now there will be a new symlink ``/usr/bin/gfortran-7`` pointing to
``/usr/bin/x86_64-linux-gnu-gfortran-7``. This ``x86_64-linux-gnu-gfortran-7`` is the 
compiler you need to compile SCONE. Now, you would want ``gfortran`` to point to this compiler. Issue the following commands, 

```
$ sudo rm /usr/bin/gfortran
$ sudo ln -s /usr/bin/x86_64-linux-gnu-gfortran-7 /usr/bin/gfortran
```

Now, you can check whether the association was correctly done by typing ``gfortran --version``. The reported version should now be 7.x.x.

Move on to installing the remaining repositories, 

```
$ sudo apt install git
$ sudo apt install libomp-dev libomp5
$ sudo apt install libblas-dev liblapack-dev
```

Installtion of pFUnit is omitted. 

## Build SCONE 

### Clone the SCONE repository

To use SCONE, first, move to your home directory ``~`` use clone the repository,

```
$ cd ~
$ git clone --branch develop https://bitbucket.org/Mikolaj_Adam_Kowalski/scone SCONE
$ cd SCONE
$ rm -rf .git/
```
This will clone only the "develop" branch of the project. 

### Configure and Compile SCONE
SCONE uses CMake to control the compilation process. To avoid clutter, you are going to create a new folder inside the SCONE directory and put the generated configuration and build files inside it. Issue the following commands,

```
$ mkdir Build
$ cd Build 
$ sudo cmake .. -DBUILD_TESTS=OFF
```
At this point, the makefiles are generated. Now compile SCONE using the following command, 

```
$ sudo make
````
With the compilation done, the executable binary "scone.out" should be located inside the same Build directory.

### Create a symlink

The following lets SCONE be invoked from anywhere (creating a symlink),

```
$ sudo ln -s ~/scone/Build/scone.out /usr/bin/scone
```

Now you can run SCONE from anywhere you want by just typing ``scone`` in the terminal. 


## Obtaining ACE library and configuring the directory file 

There are several evaluation projects, hence several outlets to get the 
evaluated nuclear cross-section data libraries from. The evaluated data libraries are distributed
as ENDF tapes. However, SCONE requires a "compact" version of the tapes as ACE-formatted 
tables. There is always an extra step involved to get from ENDF to ACE. 
Fortunately, ACE libraries based on different evaluations can be readily downloaded. 

ENDF/B ACE libraries can be downloaded from [NNDC/BNL](https://www.nndc.bnl.gov/endf/) or [LANL](https://nucleardata.lanl.gov/ace/) websites. Additionally, the JEFF ACE libraries can be downloaded from the [OACD/NEA](https://www.oecd-nea.org/dbdata/jeff/) website. 

As an example, 
head over to the [LANL](https://nucleardata.lanl.gov/ace/) website and expand the "ACE LIBRARIES" pane one left. Select "ENDF70" and download the .tgz archive. To download from the command line, do the following,

````
$ cd ~ 
$ wget https://nucleardata.lanl.gov/lib/endf70.tgz
````

Note that the URL following ``wget`` might get obsolete. In any case, you can always visit the website and get the archive/URL.  

Once downloaded, expand the archive. We shall also rename the folder (and the subfolder that actually contains the ACE files) for our convenience. Type in the following in the terminal,

````
$ tar xzf endf70.tgz
$ mv endf70 XS
$ mv XS/endf70 XS/acedata
````

Hence, the ACE libraries are located at ``~/XS/acedata/``.

Now it's time to create the cross-section directory file for SCONE. 
The directory file lists entry information of all cross-sections available in 
the ACE library. 

To create the directory file, first, download the shell script "makeXSdir.sh" provided with this repository.
Then place the script alongside your ``acedata`` folder. In the terminal, 

````
$ wget https://raw.githubusercontent.com/saad589/SCONEBuild/master/makeXSdir.sh
$ mv makeXSdir.sh ~/XS/
````

Inside ``~/XS/``, you now have ``makeXSdir.sh`` script and ``acedata`` folder. Run the ``makeXSdir.sh`` script, 

````
$ cd XS/
$ chmod +x makeXSdir.sh
$ ./makeXSdir.sh
````

This should create a directory file with the name "foo.aceXS" on the same directory. Let's give the directory file a representative name, 

````
$ mv foo.aceXS ENDFB70.aceXS
````

You are all set! You have the directory generated file as ``~/XS/ENDFB70.aceXS`` and the ACE tables located at ``~/XS/acedata/``.

## Preparing your first input file and running SCONE

SCONE input file for the STACY-30 (model 2) benchmark (ref. ICSBEB/LEU-SOL-THERM-007) is given below,

````
// STACY-30 (model 2) ref. LEU-SOL-THERM-007
// Written by saad589

type eigenPhysicsPackage;

pop      200000;
active   1000; 
inactive 100; 
XSdata   ceData;
dataType ce; 

// outputFile ./Stacy;
// printSource 1; 

collisionOperator { neutronCE {type neutronCEstd;
                               // impAbs         1;
                               // roulette       1;
                              } 
                    neutronMG {type neutronMGstd;} 
                  } 

transportOperator { 
                    type transportOperatorST;
					          cache 1;
                  } 

inactiveTally {
              } 

activeTally {  
                fluxE{ type collisionClerk;
                        map { type energyMap; grid log; min 1.0E-11; max 2.0E1; N 172;} 
                        response (fluxE); fluxE {type fluxResponse;}
                      } 

                fluxR{ type collisionClerk; 
                        map {
                          type multiMap;
                          maps (ene mat); 
                          ene {type energyMap; grid log; min 1.0E-11; max 2.0E1; N 172;} 
                          mat {type materialMap; materials (fuel ref);}
                        }
                        response (fluxR); fluxR {type fluxResponse;}
                      } 
      	    }

geometry	{ 
    type geometryStd;
    boundary (0 0 0 0 0 0);
    graph { type shrunk;}

  surfaces {
    squareBound { id 69; type zTruncCylinder; origin (0.0 0.0 75.25); halfwidth 77.25; radius 29.8;}
    surf_1	{ id 1; type zPlane; z0 0.0; }  
    surf_2	{ id 2; type zPlane; z0 46.83;  }
    surf_3	{ id 3; type zPlane; z0 150.0;  }  
    surf_10	{ id 10; type zCylinder; origin (0.0 0.0 0.0); radius 29.5; }
    }

  cells {
    cell_1	{ id 1; type simpleCell; surfaces (1 -2 -10 ); filltype mat; material mat1; }
    cell_2	{ id 2; type simpleCell; surfaces (2 -3 -10 ); filltype mat; material mat3; }
    cell_3	{ id 3; type simpleCell; surfaces (1 -3  10 ); filltype mat; material mat2; }
    cell_4	{ id 4; type simpleCell; surfaces (-1); filltype mat; material mat2; }
    cell_5	{ id 5; type simpleCell; surfaces (3 ); filltype mat; material mat2; }
    }

  universes {
    root { id 1; type rootUniverse; border 69; fill u<2>;}
    
    geom { id 2; 
           type cellUniverse; 
        // origin (0.0 0.0 0.0); 
           cells ( 1 2 3 4 5);
         }
	}   

}

viz {
  bmp {
    type bmp;
    output img;
    what material;
    centre (0.0 0.0 77.25);
    //width (25.0 25.0);
    axis x;
    res (500 500);
  }
}

nuclearData {
  handles { 
     ceData { type aceNeutronDatabase; aceLibrary ~/XS/ENDFB70.aceXS;}
  }
  materials { 
    numberOfGroups 69; 

    mat1 {  
      temp       273; 
      composition {
    	 1001.70 5.6707E-02;
         7014.70 2.9406E-03;
         8016.70 3.8084E-02;
        92234.70 6.4430E-07;
        92235.70 7.9954E-05;
        92236.70 7.9854E-08;
        92238.70 7.1216E-04;}   
        }	 

    mat2 {  
      temp       273; 
      composition {
         6000.70 4.3736e-5;
        14000.70 1.0627e-3;
        25055.70 1.1561e-3;
        15031.70 4.3170e-5;
        16000.70 2.9782e-6;
        28000.70 8.3403e-3;
        24000.70 1.6775e-2;
        26000.70 5.9421e-2;}   
        }

    mat3 {
      temp	273; 
      composition { 
         7014.70 3.9016e-5;
         8016.70 1.0409e-5;}
      } 
  }

}
````
Save the input file as STACY. 

SCONE can be run with the following arguments,
``--plot`` plots geometry  specified by a viz dictionary in the input file.
``--omp <int>`` sets the number of OpenMP threads for within-node parallel execution.
```
$ scone STACY --plot               
$ scone STACY --omp 8          
```

The STACY input file is presented above for illustrative purposes. A host of example input files is provided with the SCONE repository that you cloned earlier. They are located at ``~/scone/InputFiles/``. To run any of the provided input files, you need to provide the location of your directory file. Also, the ``.id`` part of the ``ZA.id`` under the material cards should be changed to whatever ``.id`` is available for your ACE library.

> ``.id`` in ``ZA.id`` is a mnemonic that is defined by someone who originally prepared the ACE table. ACE tables are generated from evaluated ENDF tapes with the system code NJOY. Conventionally, ``.id`` is a two-digit integer that arbitrarily corresponds to a specific temperature. For example, ``1001.70`` might correspond to Hydrogen nuclear cross-section prepared 273 K. If you have obtained someone else's ACE library, refer to the documentation to see which ``.id`` corresponds to which temperature. 

> Additionally, you might see ``ZA.id`` is followed by a letter ``c`` or something else, for instance ``1001.70c``. As a convention, the letter ``c`` means to tell you that this is a neutron cross-section table. Also, the letter ``t`` corresponds to the bound thermal scattering cross-section table.    

## Maintainers

[@saad589](https://github.com/saad589).

## Contributing

Feel free to dive in! [Open an issue](https://github.com/saad589/matcom/issues/new) or submit PRs.

## License

[GPL](LICENSE) © saad589
