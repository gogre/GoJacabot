# irgobot
AI-enhanced Go playing client/editor

![Image description](https://github.com/gogre/irgobot/blob/master/pyramidirgit.png)

irgobot, inspired by the works of Yoshio Ishida and Michael Redmond, combines a (debugged) Swim colour-map [1]  with Influencie shadow map [2] and localised life-and-death analysis by leela-zero [3], to compute groups and Ishida-type graphics [4] of their territory and influence and (coming soon, moyos and more). 

1. https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3071677
2. https://github.com/featurecat/Influencie
3. https://github.com/leela-zero/leela-zero
4. https://senseis.xmp.net/?AllAboutThickness

A prototype of irgobot is probably most easily programmed in a fork of Influencie embedded in a fork of Lizzie or Sabaki with GTP callouts to leela-zero.

Programming irgobot (or a part of it) would make a doable computer science project for people interested in AI and visualisation.

**Examples of Use**

1. [Visualising Influence and Territory](https://youtu.be/pwxiBqykHGc)
2. Inducing Shape Flaws (tba)
3. tba
4. 

#Algorithm

**1. Draw colourmap**  
Propagate colour control  
Circumscribe clusters  
Identify and mark obviously dead stones    

**2. Redraw colour-map**  

**3. Compute cluster shadows and groups**  
foreach cluster do  
...pretendboard := board  
...Put pretend stones on colour-controlled points in pretendboard   
...Call Influencie with pretendboard to get cluster shadows   
...Circumscribe group on board from pretendboard cluster+shadows  
...*a group is a set of clusters whose shadows are contiguous*

**4. Perform group life-and-death analysis**  
 foreach group do  
...make boardcopy                 
...fillup rest of zboard with black stones                         
...poke 2 eyes in rest of zboard; endofgame:= false                       
...until endofgame do
......call leela-zero(zboard) => bestmoves,endofgame
......if null(intersection(bestmoves, group)) then pass(zboard)                    
......else makebestmove in group on zboard                   
foreach point in group do                       
......if point.occupant = enemystone and not(zboard.point.occupant = enemystone)                          
......then board.point.occupant.cluster.status:= dead
...Call leela-zero  
...Identify and mark on board locally dead stones surrounded by group

**5.** Redraw colour map and shadows  

**6.** Draw graphics on board  


**Propagate colour control =**  
Until no new coloured points or links are discovered, repeat 1,2,3                    
...1. a newly-coloured point colours its links *a stone is a coloured point*                       
...2. an uncoloured empty point [edge point],                     
.........at least 3 [2] of whose links are same-coloured                    
.........and none opposite-coloured,                    
......is coloured                  
...3. an uncoloured link connecting 2 uncoloured points,                     
.........each of which has at least one coloured link                     
.........and no opposite-colored links,                    
......is coloured                 
if a link becomes coloured by both colours,                       
...its colour is neutralised.                     


**Circumscribe clusters =**                    
clusters.numberof := 0;                 
foreach point in board do                       
...if every(point.link) is same-colour then
......point.colour := colour-controlled
...if every(point.link) is same-colour or neutral then                     
......foreach (point.link.otherpoint) do
......... *ie the point at the other end of the link* 
......if member(point.link.otherpoint, cluster)                  
......then add(point, point.link.otherpoint.cluster)
...if member(point, clusterA) and member(point, clusterB) then unite(clusterA,clusterB)
......else makenewcluster(point)  

**Identify and mark obviously dead stones =**                
...foreach cluster in clusters do               
......identify(cluster.eyes);                
......if number(cluster.eyes) > 2                 
.........or size(cluster.eyes) > 3               
.........and shape(cluster.eye) not(in{dead-shapes})                  
......then cluster.status := alive                  
......elsif surrounded(cluster, enemies)                
............and foreach enemy in enemies (enemy.lad = alive)                      
......then cluster.status := dead

**identify(cluster.eyes) =**
foreach point in cluster do
...if colour-controlled(point) and
......not border(point) or stone(point)
...then append(point, cluster.eyes) 

**surrounded (cluster, enemies) =**          
not(forany point in border(cluster) 
...path(friend(point)
...or path(openspace, point))

**makenewcluster(point) =**    
..clusters.numberof +:= 1;   
...let newcluster = ({point}, clusters.numberof)   
...paint(board.point, point.newcluster.number, point.colour(point)   

**Circumscribe group on board from pretendboard clusters+shadows =**
iboard := board;                        
foreach cluster in board do                      
...foreach point in cluster do                       
......if point is coloured then iboard.point := pretendstone(colour)                       
...isboard:= Influencie (iboard)                              
...foreach point in board do point.colour:= isboard.point.shadow;                            
                       

                        

