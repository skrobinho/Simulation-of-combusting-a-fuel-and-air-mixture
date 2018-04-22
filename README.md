# Studia

import cantera as ct
import numpy as np
import matplotlib.pyplot as plt
import math

# Symulation of combusting a fuel and air mixture in variable volume(piston engine) with spontaneous ignition

# Three kinds of fuel can be examined - methane, propane, butane
fuel = input("Enter a type of fuel (CH4/C3H8/C4H10):")
fuel = fuel.lower()


# gas1 - mixture of fuel and air
fuel_air = ct.Solution('gri30.xml')

if fuel == 'ch4':
    fuel_air.X = 'CH4:1.0, O2:2.0, N2:7.52'
    c = True
elif fuel == 'c3h8':
    fuel_air.X = 'C3H8:1.0, O2:5.0, N2:18.8'
    c = True
elif fuel == 'c4h10':
    fuel_air.X = 'C4H10:1.0, O2:6.5, N2:24.44'
    c = True
else:
    print ("You entered a wrong type of fuel")
    c = False
       
        
if c == True:
    
# Initial temperature of reactor containing fuel    
    x = input("Enter the temperature of the reactor[K]:")
    x = float(x)
    
    fuel_air.TP = x, ct.one_atm
    
    y = input("Enter the time interval[s]:")
    y = float(y)
    
    print ("\n")
    
    # gas2 - air
    air = ct.Solution('gri30.xml')
    air.TPX = 300.15, ct.one_atm, 'O2:1.0, N2:3.76'


    # reactor1 - combustor
    combustor = ct.Reactor(fuel_air, volume=0.5)

    # reactor2 - free space below piston
    r2 = ct.Reactor(air, volume=0.5)

    # wall - piston
    piston = ct.Wall(combustor,r2, K = 1e2)

    simulation = ct.ReactorNet([combustor, r2])
    
    time = []
    temperature = []
    pressure = []
    
    for n in range(60):
        t = (n+1)*y
        simulation.advance(t)
        time.append(t)
        temperature.append(fuel_air.T)
        pressure.append(fuel_air.P)
        print ("%f\t %f\t %f" %(t, fuel_air.T, fuel_air.P))
        
    physical_quantities = {'Temperature': temperature, 'Pressure': pressure }

    for n in physical_quantities:
        plt.plot(time, physical_quantities[n])
        plt.xlabel('time')
        plt.ylabel(n)
        plt.title("{}(time)".format(n))
        plt.savefig('{} graph.png'.format(n))
        plt.close()
        
    time1 = 100000.
    simulation.advance(time1)

# Condition if spontaneous ignition has happend     
    if combustor.T < 2000.:        
        print ("Samozapłon nie nastąpił")
    else:
        print ("Samozapłon nastąpił")