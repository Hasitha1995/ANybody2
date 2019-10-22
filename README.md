#include "../libdef.any"

// --------------------------------------------------------
// This is a model comprising the full body model, a chair
// and an interface between them.
// The interface is made such that the posture of the body
// is dependent of the settings if the chair.
// The settings of the chair (and thereby the posture of 
// the body) are controlled by the parameters in the 
// folders DrvPos and DrvVel.
// The support between the body and the chair is made with artificial muscles
// which also handles friction
// --------------------------------------------------------

Main = {
  // ----------------------------------------------------------
  // Model parameters: Driver position and support settings
  // ----------------------------------------------------------
  #include "Model/InputParameters.any"
  
  // ----------------------------------------------------------
  // Path to draw settings
  // ----------------------------------------------------------
  #path BM_DRAWSETTINGS_FILE "Model/DrawSettings.any"

  // ----------------------------------------------------------
  // Model configuration:
  // For more info on body model configuration options please load the model and double click on: 
  // #path HTML_DOC "<AMMR_PATH_DOC>/bm_config/index.html"
  // ----------------------------------------------------------

  // Original application was created using previous leg model
  #define BM_LEG_MODEL _LEG_MODEL_Leg_
  #define BM_SCALING _SCALING_NONE_

  // Switch on neck
  #define BM_TRUNK_NECK_MUSCLES _MUSCLES_SIMPLE_
  
  // The Mannequin file specifies load-time positions for all the segments
  // in the HumanModel. This is important for the model's ablity to resolve
  // all the kinematic constraints when the model is analyzed.
  // The mannequin file also drives those degrees of freedom of the human 
  // model that are not governed by problem-specific drivers at run time.    
  #path BM_MANNEQUIN_FILE "Model/Mannequin.any" 

  // Model of the human body to be used for the analysis  
  #include "<ANYBODY_PATH_BODY>/HumanModel.any"

  /// The actual model where all components are assembled  
  AnyFolder Model ={
    /// Body model without default drivers
    AnyFolder &HumanModel=.HumanModel.BodyModel;
    /// Reference to the mannequin folder (used by drivers)
    AnyFolder &Mannequin =.HumanModel.Mannequin;  
 
    #include "Model/MannequinValuesFromModel.any"   
    
    AnyFolder EnvironmentModel = {
      #include "Model/Environment.any"
    };
    
    AnyFolder ModelEnvironmentConnection = {
      #include "Model/ConnectionSegments.any"
      #include "Model/JointsAndDrivers.any"
      
      AnyFolder &RefPM=Main.Model.EnvironmentModel;
      #include "Model/InitialPositionsEnvironment.any" 
      #include "Model/Measures.any"
      #include "Model/Support.any" 
    }; // ModelEnvironmentConnection
    
    
    // -------------------------------------------------------------
    
  }; // Model
  
  // --------------------------------------------------------------
  // Studies
  // --------------------------------------------------------------
  AnyBodyStudy Study = {
    AnyFolder &Model = .Model;
   
    
    tEnd = 1;
    Gravity = {0.0, -9.81, 0.0};
    nStep = 10;
    
    //Output from the Vastus Muscles
    #if BM_TRUNK_NECK_MUSCLES == _MUSCLES_SIMPLE_
    AnyVar VastiAct = 
    max({Main.Model.HumanModel.Right.Leg.Mus.VastusMedialis.Activity,
      Main.Model.HumanModel.Right.Leg.Mus.VastusIntermedius.Activity,
      Main.Model.HumanModel.Right.Leg.Mus.VastusLateralis.Activity});
    
    //Size of total force in the L5 Sacrum joint
    AnyVar L5SacrumReac = (Main.Model.HumanModel.Trunk.JointsLumbar.L5SacrumJnt.Constraints.Reaction.Fout[0]^2+
    Main.Model.HumanModel.Trunk.JointsLumbar.L5SacrumJnt.Constraints.Reaction.Fout[1]^2+
    Main.Model.HumanModel.Trunk.JointsLumbar.L5SacrumJnt.Constraints.Reaction.Fout[2]^2)^0.5;
    
    
    AnyOutputFun MaxAct = {
      Val = .MaxMuscleActivity;
    };    
    AnyOutputFun MaxVastiAct = {
      Val = .VastiAct;
    };
    AnyOutputFun L5SacrumR = {
      Val = .L5SacrumReac;
    }; 
    #endif
  }; // Study
  
  // Include an operation sequence to run all required steps of your application (see Operations tab)
    #include "<ANYBODY_PATH_MODELUTILS>\Operations\RunAppSequence.any"    
    
  
};// Main
















