# Citcoms-3.3.1_Regional_weak_layer
This CitcomS version is used to introduce regional weak zones in colder regions (slab).
For modifications see Viscosity_structures.C


Modifications:

This is are changes the changes that I included, which are to be read from input file.

	/* Added by Debanjan Pal */
	input_int("reg_weak_layer",&(E->viscosity.reg_weak_layer),"1",m);
	input_int("weak_layer_no",&(E->viscosity.weak_layer_no),"1",m);
        input_float("weak_layer_ff",&(E->viscosity.weak_layer_ff),"0.01",m);

	/*************************/
############ INPUT FILE READ IN PARAMETERS
*****	reg_weak_layer=off/on (whether the regional weak layer will be present/absent)
	weak_layer_no= give the mat_layer no in which the weak layer to be generated
 	weak_layer_ff= viscosity pre-factor in the weak layer.

The regional weak layer only now  works for rheol=3, but can be added to other cases, example shown below:


	   /* consistent handling : l is (material number - 1) to allow
       addressing viscosity arrays, which are all 0...n-1  */
    switch (E->viscosity.RHEOL)   {
     case 1:
        /* eta = N_0 exp( E * (T_0 - T))  */
        for(m=1;m<=E->sphere.caps_per_proc;m++)
            for(i=1;i<=nel;i++)   {
                l = E->mat[m][i] - 1;

                if(E->control.mat_control==0)
                    tempa = E->viscosity.N0[l];
                else 
                    tempa = E->viscosity.N0[l]*E->VIP[m][i];

                for(kk=1;kk<=ends;kk++) {
                    TT[kk] = E->T[m][E->ien[m][i].node[kk]];
                }

                for(jj=1;jj<=vpts;jj++) {
                    temp=0.0;
                    for(kk=1;kk<=ends;kk++)   {
                        temp += TT[kk] * E->N.vpt[GNVINDEX(kk,jj)];
                    }
/* Modified by Debanjan Pal to introduce regional weak layer below 660 when there is slab */

		 /* layer number n-1 */
		 if (E->viscosity.reg_weak_layer==1) {
				layer_no = E->viscosity.weak_layer_no - 1;
			if(l == layer_no)                       
				if((temp - 0.5) < -0.1) 

					  EEta[m][ (i-1)*vpts + jj ] = tempa*E->viscosity.weak_layer_ff*
                        exp( E->viscosity.E[l] * (E->viscosity.T[l] - temp));

				else

					 EEta[m][ (i-1)*vpts + jj ] = tempa*
                        exp( E->viscosity.E[l] * (E->viscosity.T[l] - temp));
			
			else

                    EEta[m][ (i-1)*vpts + jj ] = tempa*
                        exp( E->viscosity.E[l] * (E->viscosity.T[l] - temp));
	
	}
			 	else {
				
				 EEta[m][ (i-1)*vpts + jj ] = tempa*
                        exp( E->viscosity.E[l] * (E->viscosity.T[l] - temp)); 
		   
		   }
                }
            }
        break;


This are the changes in Instructions.c 

fprintf(fp, "reg_weak_layer=%d\n", E->viscosity.reg_weak_layer); /* Added by Debanjan Pal*/
     fprintf(fp, "weak_layer_no=%d\n", E->viscosity.weak_layer_no);  /* Added by Debanjan Pal*/
      fprintf(fp, "weak_layer_ff=%d\n", E->viscosity.weak_layer_ff);  /* Added by Debanjan Pal*/




