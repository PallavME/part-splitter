# part-splitter

# this is hypermesh TCL code for part splitting and matching component by surface number


###################
    # CREATING WIDGETS
    ###################
	
	set top .window
    catch {destroy $top}
    toplevel $top -class Toplevel
    wm focusmodel $top passive
    wm geometry $top 190x220+719+543; update
    wm maxsize $top 190 220
    wm minsize $top 115 1
    wm overrideredirect $top 0
    wm resizable $top 1 1
    wm deiconify $top
    wm title $top "temp"
	
	
	button $top.but46 \
        -pady 0 -text "Delete solids" -command {delsolid}
    button $top.but47 \
        -pady 0 -text "select surface to segregate" -command {partseg}
    button $top.but49 \
        -pady 0 -text "Match parts" -command {matchpart}
		
	    # SETTING GEOMETRY
    ###################
    place $top.but46 \
        -in $top -x 30 -y 15 -width 150 -height 30 -anchor nw \
        -bordermode ignore 
    place $top.but47 \
        -in $top -x 30 -y 55 -width 150 -height 30 -anchor nw \
        -bordermode ignore 
    place $top.but49 \
        -in $top -x 30 -y 95 -width 150 -height 30 -anchor nw \
        -bordermode ignore 	
		
		
		
		
 proc delsolid {} {	
   #alwys delete all solids if you are running on intial cad

    *createmark solids 1 all
    *deletesolidswithelems 1 1 1
  }





 proc partseg {} {
      #cleaning all mark buckets
      *clearmark surfs 1
      *clearmark surfs 2
     *clearmark elems 1
     *clearmark elems 2
     *clearmark comps 1
     *clearmark comps 2


#select all surfaces (variable surfsall for all selected surfaces)

       *createmarkpanel surfs 1
       set surfsall [hm_getmark surfs 1] 

#move all selected surfaces into temp collector named "TEMP_1"

       *createentity comps cardimage=Part includeid=0 name="TEMP_1"
       *createmark components 1 "TEMP_1"
       *movemark surfs 1 "TEMP_1"
   
#eval means evaluate command 
#"eval *createmark surfs 1 "by comps" "TEMP_1" " by this condition we can update list after each loop
#this loop will work till no surface remain in "TEMP_1" collector 
   
       while  {[lindex [eval *createmark surfs 1 "by comps" "TEMP_1"] 0] !=0} {
   
           *createmark surfs 1 "by comps" "TEMP_1"  
           set singlesurf [hm_getmark surfs 1]
	   *clearmark surf 1
        
         # variable firstsurf used to get 1st surface id of each loop update

	   set firstsurf [lindex $singlesurf 0]
        
         # by appendmark getting all surface to attaced the 1st surface  
 
          *createmark surfs 1 $firstsurf
	  *appendmark surfs 1 "by attached"
          set attachsurf [hm_getmark surfs 1]

	# creatin a component with firstsurf id and move all attached surface

           *createentity comps cardimage=Part includeid=0 name="surf_$firstsurf"
           *createmark components 1 "surf_$firstsurf"
	       *movemark surfs 1 "surf_$firstsurf"
		   
    #move to assembly
	        if { [hm_entityinfo exist assems "SEGREGATED_CAD"] == 0}  {
	           *createentity assems cardimage=SUBSET includeid=0 name=SEGREGATED_CAD
		    }
			   *createmark assembly 1 "SEGREGATED_CAD"
			   set k [hm_getmark assems 1]
               *createmark components 1 "surf_$firstsurf" 
               *assemblyaddmark $k components 1

      #now in "TEMP_1" components surfaces will again evaluate in loop as discussed in above

	 }
	
	
	
  }
	





proc matchpart {} {


         *createmark comps 1 "TEMP_1" "surf_"
	     *deletemark comps 1
		 *clearmark comps 1

		 *createmark comps 1 "by assems" "SEGREGATED_CAD"
		 set allcomps [hm_getmark comps 1]
		 set noc [llength $allcomps]
		  
		 
		 
		 #check by number of surfaces compaire
		 
		
		
		 
		 foreach compsid $allcomps  {
		    *createmark comps 1 "by assems" "SEGREGATED_CAD"
			set remcomps [hm_getmark comps 1]
			*createmark surfs 1 "by comps" $compsid
			set surfsetmain  [hm_getmark surfs 1]
			set nosurfmain [llength $surfsetmain ]
			
			foreach remcompid $remcomps {
			  
			 *createmark surfs 1 "by comps" $remcompid
			 set surfsetcheck  [hm_getmark surfs 1]
			 set nosurfcheck [llength $surfsetcheck ]
			    
			  if { $nosurfmain == $nosurfcheck } { 
				   puts "surface matched"
			
			       if { [hm_entityinfo exist assems "MAtch_$compsid"] == 0} {
	                 *createentity assems cardimage=SUBSET includeid=0 name="MAtch_$compsid"
		            }
			  
			         *createmark assembly 1 -1
					 set j [hm_getmark assems 1]
			   
			         *createmark assemblies 2 "SEGREGATED_CAD"
					 *createmark components 1 $remcompid
                     *assemblyremovemark 2 components 1
			         
					 *createmark components 3 $remcompid
                     *assemblyaddmark $j components 3
					 
					 
                }		  
			  
			}   
		  }	
}
