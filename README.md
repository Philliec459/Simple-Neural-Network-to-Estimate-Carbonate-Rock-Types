# Simple-Neural-Network-to-Estimate-Rock-Types-Rock-Types
We have developed a simple, 1 layer neural network to estimate Macro, Meso and Micro Rock Types
### Introduction
#### Predict Macro, Meso and Micro Rock Types (RT)
The objective of this project is to estimate Macro, Meso and Micro Rock Types (RTs). In this repository we are using a simple this single layer neural network to predict our RT. Therefore, we modified the Petrophysical Rock Types (PRT) as defined by Clerke(1) for the Arab-D carbonate reservoir. The objective is to use just Porosity and Permeability to estimate our Macro, Meso or Micro RT.

The Arab D data set published by Clerke is quite distinctive. Clerke acquired nearly 450 High Pressure Mercury Injection Capillary Pressure (HPMI) measurements in the Arab D reservoir; however, Clerke's final samples were randomly selected from 1,000's of pre-qualified core samples ensuring a broad distribution and representation of all Petrophysical properties. This created one of the best Core Analysis datasets every collected in our industry. 

Clerke began evaluating this dataset by fitting a Thomeer hyperbolas for each pore system in each sample to generate the published Thomeer Capillary Pressure parameters. From these data Clerke established his Petrophysical Rock Types (PRT) based on the Initial Displacement Pressures for each pore system and the number of pore systems present in each sample. From the figure below it is rather evident that Clerke's PRTs are Petrophysically well-defined in poro-perm space where each color represents a different PRT.  The Capillary Pressure curves and Pore Throat Distributions (PTD) shown on the right hand side of the figure illustrate the unique characteristics of each PRT. 

![TS_Image](PRT.png)

###### The characterization of Clerke's PRTs are shown below:

![TS_Image](Rock-Types.png)

As can be seen in the first figure above, the PRTs are rather well segregated in the Porosity vs. Permeability Cross Plot as they fall in distinct regions or clusters on the Cross Plot.For modeling purposes it is important to take advantage of the excellent correlations between the PRTs. 

The first part of this notebook develops our single layer neural network as inspired by the video series from giant_neural_network on YouTube. 

https://www.youtube.com/watch?v=LSr96IZQknc

We have used Clerke's Rosetta Stone data and his PRTs as our training set, except that we combined all the macros PRTs into one RT that had a value of 2. We combined all the Type 1 Meso PRT into a RT with a value of 1 and all the Micro PRT compose our third RT with a value of 0. The following is the Sigmoid s-curve that we are using

![TS_Image](sigmoid.png)

We have expanded our Sigmoid curve for values from 0 to 2 to accomodate our 3 RTs at 0, 1 and 2.
 
        def sigmoid(x):
            return 2/(1 + np.exp(-x))

We first calculate a variable z which is a function if weights and bias:
        
        z = Porosity * weight1 + permeability * weight2 + bias

Our prediction (pred) is then a function of sigmoid:        
        
        pred = sigmoid(z) 

where sigmoid is defined by:
        
        def sigmoid(x):
            return 2/(1 + np.exp(-x))

The term z is actually the x-axis on the above sigmoid curve, and y is a function of sigmoid(z). 

For training we make about 1,000 iterations to optimize on the weights (w1 and w2) and bias for our single layer. These weights can be saved and then used in future projects to estimate RT from the user's input of Porosity and log10 of Permeability. We can calculates the most probable Rock Type and even provide Capillary Pressure curves for each Rock Type.

The following is the training loop. 

     # Start the training loop
     learning_rate = 0.1
     costs = []
     cost_all = []

     w1 = np.random.randn()
     w2 = np.random.randn()
     b  = np.random.randn()

     # Save weights from previous run. These can be loaded to continue from there
     #w1 = 21.5
     #w2 = 10.5
     #b  = -77

     for i in range(1000):
         ri = np.random.randint(len(data))
         point = data[ri]

         z = point[0]*w1 + point[1]*w2 + b
         pred = sigmoid(z)

         target = point[2]
         cost = np.square(pred - target)

         cost_all.append(cost)

         ###costs.append(cost)

         #derivatives
         dcost_pred = 2 * (pred - target)
         dpred_dz   = sigmoid_p(z)
         dz_dw1 = point[0]
         dz_dw2 = point[1]
         dz_db  = 1

         dcost_dw1 = dcost_pred * dpred_dz * dz_dw1
         dcost_dw2 = dcost_pred * dpred_dz * dz_dw2
         dcost_db  = dcost_pred * dpred_dz * dz_db

         w1 = w1 - learning_rate * dcost_dw1
         w2 = w2 - learning_rate * dcost_dw2
         b  = b  - learning_rate * dcost_db

         if i % 100==0:
             cost_sum = 0
             for j in range(len(data)):
                 point = data[j]

                 z = point[0]*w1 + point[1]*w2 + b
                 pred = sigmoid(z)

                 target=point[2]
                 cost_sum += np.square(pred - target)

             costs.append(cost_sum/len(data))
             #costs.append(cost_sum)


To make a prediction we would use code similar to the following where we first compute z based on the product of our inputs and weights:

     for i in range(len(data)):
         point = data[i]
         z = point[0] * w1 + point[1]* w2 + b
         pred = sigmoid(z)
         print("PRT:", point[2],",", "pred: {}".format(pred))

and then we predict where we are in the sigmoid curve scaled from 0 to 2 (pred = sigmoid(z).  

The final RT are then defined as shown below:
        
        if pred > 1.56:
            RT=2
        elif pred < 0.25:
            RT=0
        else:
            RT=1
            
![TS_Image](pred.png)

For nearly 300 samples, we have accurately predicted the correct RT for all by 6-7 samples using the RT cutoffs shown above. The cutoffs can be varied a bit to improve upon our RT estimations. As they say, simplicity is the best design. 


1 Clerke, E. A., Mueller III, H. W., Phillips, E. C., Eyvazzadeh, R. Y., Jones, D. H., Ramamoorthy, R., Srivastava, A., (2008) “Application of Thomeer Hyperbolas to decode the pore systems, facies and reservoir properties of the Upper Jurassic Arab D Limestone, Ghawar field, Saudi Arabia: A Rosetta Stone approach”, GeoArabia, Vol. 13, No. 4, p. 113-160, October, 2008. 

