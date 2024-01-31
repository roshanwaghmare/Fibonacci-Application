# Multi Container Application Fibonacci-Application

Section : 08
We are creating application that calculate Fibonacci value when we enter index numbers eg: 1,2,3,4 etc 

![Screenshot 2024-01-29 133711](https://github.com/roshanwaghmare/Fibonacci-Application/assets/142305817/b164bdf6-d39d-440e-93ea-ef75650c9ec1)

## Steps


## to open file and folder directly form terminal u need to use below cmd

```
sudo apt install nautilus
xdg-open .
open .
```
this setup is not working for us so we find below solution 

1 now we have to get all the java code from our local machine to ubuntu terminal so use below

````
Open .

was not working in my ubuntu so use below path
cd .. till main directory
cd mnt/
i ahev copied complex folder from downloads and paste in the c dirve 
cp -r complex/ /home/roshanw/
````
## hopefully you now have a complex directory and inside there is the client server in worker folders  each of those folders represent the React server the express API and the worker process as well. We're now not gonna start the process of adding Docker containers to each of these applications.

We can start each of them up inside of a development environment. We're gonna take the React project, the express API and the worker as well and we're going to make Dev Docker files for each one.

## now will create three dockerfile for each React server the express API and the worker { Dev Environment}

at the project files inside of each of those directories and inside of each of them we're gonna set up a pretty similar Docker file workflow. Remember that each one of these projects is probably gonna have a package.json file that records all of the dependencies of our project. We're going to copy over that package.json file as step number one. We're then going to run a NPM install and then we'll copy over everything else. The last thing to keep in mind is that we're gonna set up a Docker composed file and that Docker compose is gonna set up volumes for each of these projects so that we kind of share all of the source code
**inside of each project and that's what's gonna make sure that we don't have to rebuild our image entirely from scratch every time that we make one tiny little change.**
