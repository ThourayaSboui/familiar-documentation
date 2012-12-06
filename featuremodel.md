# Feature Models 

## Specification

Multiple notations are supported for specifying feature models. 
It means you can: 
 * load feature models in those notations (using '''FM''' constructor) ;
 * serialize the feature models in those notations (using '''save''').

### Formats

The following formats are fully supported:
 * [SPLOT] (http://www.splot-research.org/), 
 * [FeatureIDE] (http://wwwiti.cs.uni-magdeburg.de/iti_db/research/featureide/), 
 * a subset of [TVL] (http://www.info.fundp.ac.be/~acs/tvl/), 
 * S2T2
 * an internal notation (see below)


## Internal notation

fm1 = FM (A : B C [D]; D : (E|F|G); C : (H|I|J)+ ; (!C | I) ; )

 * A is the root 
 * B, C and D are child-features of A: B and C are mandatory whereas D is optional 
 * E, F and G form a Xor-group and are child-features of D 
 * H, I, and J form an Or-group and are child-features of C 
 * (!C | I) is equivalent to (C -> I) and is an internal constraint of the feature model

Here is a diagrammatic representation of fm1 with FeatureIDE editor: 


[[Image(wiki:fmload:fm1-FeatureIDE.png, 20%)]]  


See the grammar below:

{{{
FeatureModel : 'FM' LEFT_PAREN  (features+=Production ';')+ ( expr+=Constraint ';')* ) RIGHT_PAREN ;
Production  : name=FML_IDENTIFIER ':' features+=Child ;

Child       : (Mandatory
                | Optional
                | Mutexgroup 
                | Xorgroup
                | Orgroup) ;

Mandatory   : name=ID ;
Optional    : '[' name=FML_IDENTIFIER ']' ;
Mutexgroup   : LEFT_PAREN features+=ID  ('|' features+=ID)+ RIGHT_PAREN '?' ;
Xorgroup   : LEFT_PAREN features+=ID  ('|' features+=ID)+ RIGHT_PAREN ;
Orgroup    : LEFT_PAREN features+=ID  ('|' features+=ID)+ RIGHT_PAREN '+' ;


ID : ('^')?('a'..'z'|'A'..'Z'|'_') ('a'..'z'|'A'..'Z'|'_'|'0'..'9')*; 

Constraint       : Constraint  ('|' | '&' | '->' | '<->') Constraint
            | '!' Constraint 
            | '(' Constraint ')'
            | ID
}}}

== Importing feature models in other notation == 

You can also import feature models in other notation. 
This time, you need to specify a filename (using quotes). 

'''FM''' ("filename")

At the moment, the filename extension is important (see below for the corresponding between extension and format) 

[[Image(wiki:fmload:FML-interoperability.png, 20%)]]

For example,

fm2 = FM ("internal1.m")

assigns to fm2 a feature model which has been created using FeatureIDE notation and file with name "internal1.m". 


=== Formats currently supported ===

See https://nyx.unice.fr/projects/familiar/milestone/Interoperability%3A%20convert%20in%20different%20formats (you need a login)


= Serializing Feature Models = 

You can serialize feature models in different formats. 

First, the '''convert''' returns a string representation of the feature model in the specified format.
{{{
fml> convert fm1 in featureide
res0: (STRING) A : [D] B C+ :: _A ;

D : E
	| G
	| F ;

C : J
	| I
	| H ;

%%

C implies I ;
}}}


'''save''' serializes the content of convert, e.g., (convert fm1 in featureide), in a file (by default, in the "output" folder of the Eclipse project in which the FAMILIAR script is executed or in the standalone version, in the "output" folder where the jar is located). 



= Other facilities to interoperate =

 * Loading
  * You can implement a bridge between your formalism and one of the formalism supported by FAMILIAR 
   * Hint: it can be a model-to-text transformation or a model-to-model transformation (using the Ecore metamodel generated by Xtext)
  * Note that FAMILIAR can load a feature model serialized in an XMI file conforms to the Ecore metamodel generated by Xtext. 
  * You can use the class FMLFeatureModelReader.java


{{{
FeatureModel fm = new FMLFeatureModelReader().parseString("FM (A: B C [D];) ");
FeatureModel fm2 = new FMLFeatureModelReader().parseFile(new File("examples/testing/FMs/fm2.fml"));
XMIResource xmi = new FMLFeatureModelWriter(fm2).toXMI("examples/testing/FMs/fm2ecore");
FeatureModel fm3 = new FMLFeatureModelReader().parseXMIFile(xmi);
}}}


 
 * Serializing 
  * Once a feature model is serialized in an XMI file conforms to the Ecore metamodel generated by Xtext, you can use state-of-the-art modeling tools to visit the model. 
  * Or simply using the class FMLFeatureModelWriter.java

{{{
// serializing fm2 to XMI
XMIResource xmi = new FMLFeatureModelWriter(fm2).toXMI("examples/testing/FMs/fm2ecore");
// or to text (String representation)
new FMLFeatureModelWriter(fm2).toString() ; 
}}}

  * Or extending the abstract class FeatureModelVisitor.java

{{{
public abstract class FeatureModelVisitor {
	
	/*
	 *  the feature model to visit
	 */
	
	protected FeatureModel fm ;
	
	public FeatureModelVisitor(FeatureModel fm) {
		this.fm = fm ;
	}
	
	/*
	 * Entry point
	 */
	
	public abstract String treatFeatureModel(FeatureModel fm) ; 
	
	/*
	 * Treat a production (like a grammar production in GUIDSL)
	 */
	
	public abstract String treatProd(Production prod) ;

	/*
	 * Treat a child feature (Xor, Or, And)
	 */
	
	public abstract String treatChild(Child c) ;

	/*
	 * Treat a constraint (e.g., A implies B)
	 * 
	 */
	
	public abstract String treatConstraint(CNF constraint) ;
	
}

}}}






/******** FEATURE MODEL **************/
FeatureModel : ('FM'|'featuremodel') LEFT_PAREN
                                    (
                                    (
                                    (root=ID ';')|((features+=Production ';')+ 
                                    (expr+=CNF ';')*)
                                    )
                                    | (file=StringExpr) )
                                    RIGHT_PAREN ;

//expr+=Fexpr
//FeatureDescription : Production | Expr ;

Production  : name=ID ':' features+=Child+ ;

Child       : (Mandatory
                | Optional
                | Xorgroup
                | Orgroup  
//                );             
               | Mutexgroup ) ;
                
//                | Andgroup)  ;

Mandatory   : name=FT_ID ;

//Optional    : name=FML_IDENTIFIER '?' | '[' name=FML_IDENTIFIER ']' ;

Optional    : LEFT_HOOK name=ID RIGHT_HOOK ;
Xorgroup   : LEFT_PAREN features+=FT_ID  (B_OR features+=FT_ID)+ RIGHT_PAREN ;
Orgroup    : LEFT_PAREN features+=FT_ID  (B_OR features+=FT_ID)+ RIGHT_PAREN PLUS ;
// Andgroup   : LEFT_PAREN features+=FT_ID ('|' features+=FT_ID)+ RIGHT_PAREN ; // TODO: (identifier=FML_IDENTIFIER '=')?

Mutexgroup : LEFT_PAREN features+=FT_ID  (B_OR features+=FT_ID)+ ')?' ;

terminal B_OR :     '|' | 'or'  ; 

Note this syntax
fml> fm1 = FM (A : (B or C); )
fm1: (FEATURE_MODEL) A: (B|C) ;


## Editing facilities

// change the variability operator associated to a feature
ModifyVOperator: (MandatoryEdit | OptionalEdit | AlternativeEdit | OrEdit); 

MandatoryEdit : 'setMandatory' ft=FTCommand ;
OptionalEdit : 'setOptional' ft=FTCommand ;
AlternativeEdit : 'setAlternative' fts=SetCommand ; // should be a set of features
OrEdit : 'setOr' fts=SetCommand ; // should be a set of features 


//FeatureVariabilityOperator : 'OP' LEFT_PAREN val=FeatureEdgeKind RIGHT_PAREN ;
FeatureVariabilityOperator : val=FeatureEdgeKind ;
enum FeatureEdgeKind : MANDATORY='mand'| OPTIONAL='opt'| ALTERNATIVE='Xor'| OR='Or'|MUTEX='Mutex' ;

fml> fm3 = copy fm2
fm3: (FEATURE_MODEL) A: (B|E|C|F)+ ;
fml> setAlternative fm3.A.*
res2: (BOOLEAN) true
fml> fm2
fm2: (FEATURE_MODEL) A: (B|E|C|F)+ ;
fml> fm3
fm3: (FEATURE_MODEL) A: (B|E|C|F) ;

fml> setMandatory fm3.B
res3: (BOOLEAN) true
fml> fm3 
fm3: (FEATURE_MODEL) A: B [E] [C] [F] ; 
(B -> A);
(A -> B);
fml> setOr { fm3.E fm3.C fm3.F }
res4: (BOOLEAN) true
fml> fm3
fm3: (FEATURE_MODEL) A: (E|C|F)+ B ; 
(B -> A);
(A -> B);

## Editing (2)

FeatureModelOperation : Insert | EditOperation | Extract ;
EditOperation : (RemoveFeature|RenameFeature) ;
Insert : 'insert' aspectfm=FMCommand 'into' baseft=FTCommand 'with' op=VariabilityOpCommand ; //TODO:  op2=(FML_IDENTIFIER)?  ;
RemoveFeature : 'removeFeature' feature=FTCommand ;
RenameFeature : 'renameFeature' feature=FTCommand 'as' featureNew=(StrCommand) ; //'in' fm=FML_IDENTIFIER ;
Extract: 'extract' rootfeature=FTCommand ;


## Serialization


// convert e.g., featureide, pure::variants, etc.

Convert: 'convert' v=FMCommand 'into' format=FMFormat ; // returns a string
enum FMFormat : FMLBDD='fmlbdd'|FIDE='featureide'|FCALC='fmcalc'|FFML='fml'|FSPLOT='SPLOT'|FTVL='TVL'|FTRISKELL='fd'|FFML2='xmi'|S2T2='S2T2' ;
FMLSave : ('save'|'serialize') v=FMCommand 'into' format=FMFormat;

## Automated analysis

Cores : 'cores' fm=FMCommand ; // core features
Deads : 'deads' fm=FMCommand ; // dead features
// TODO: make the difference!
FullMandatorys : ('fullMandatorys'|'falseOptionals') fm=FMCommand ; // full mandatory features

Cliques : 'cliques' fm=FMCommand ; // cliques (aka atomic sets?)

// isDead? isCore? isFullMandatory? can be written with an FML script


AnalysisOperation :
    op=('isValid' // validity of a FM
    | 'counting'  // number of products of a FM
    | 'configs' // set of products of a FM
    | 'nbFeatures' // number of features
    | 'root' // return the root feature of the fm
    | 'features' // return the set of features
    ) fm=(FMCommand|ConfigurationCommand)
    ;


Compare :
    'compare' fm_left=FMCommand fm_right=FMCommand;    // Boolean formula?

enum EditConstant : REFACTORING | SPECIALIZATION | GENERALIZATION | ARBITRARY ;

## Metrics

CTCRCommand : 'ctcr' fm=FMCommand ; 

## Composition


/****** COMPOSITION OPERATORS ***********/


Merge: 'merge' mode=MergeMode 
            ((LEFT_BRACKET
                (lfms+=FMCommand)+
            RIGHT_BRACKET) 
             | fms=LFMArgs) ;
            //(pre=Predirectives)?
            //(post=Postdirectives)?; // alignment directives

enum MergeMode : CROSS='crossproduct'|UNION='union'|SUNION='sunion'|INTER='intersection'|DIFF='diff' ;
LFMArgs : lfms+=FMCommand (COMMA lfms+=FMCommand)* ;

AggregateMerge: 'aggregateMerge' mode=MergeMode 
            ((LEFT_BRACKET
                (lfms+=FMCommand)+
            RIGHT_BRACKET) 
             | fms=LFMArgs) ;

Aggregate : 'aggregate' ((LEFT_BRACKET
					 (fms+=FMCommand)+
								 RIGHT_BRACKET) 
												 | sfms=IdentifierExpr) ('withMapping' mapping=SetCommand)?;


## Decomposition

   

Slice : 'slice' fm=FMCommand mode=SliceMode fts=SetCommand ; // issue: side effect or purely functional?

enum SliceMode : INCLUDING='including' | EXCLUDING='excluding' ;


## Misc

PairwiseCommand : 'pw' fm=FMCommand ('minimization=' minimization=IntegerCommand)? ('partial=' partial=IntegerCommand)? ;

ksynthesis
AsFM : 'asFM' conf=ConfigurationCommand;
CleanUp : 'cleanup' fm=FMCommand ; // functional style?