import rospy
from geometry_msgs.msg import Twist
from turtlesim.msg import Pose
from turtlesim.srv import Spawn

flag=[[0,0] for i in range(16)] #1-bot 
posesx=[i for i in range(16)]
posesy=[i for i in range(16)]
velx=[2]+[0 for i in range(15)]
vely=[1]+[0 for i in range(15)]

class update_callback():
    def __init__(self,index):
        self.index=index
    def update_pose(self, data):
        posesx[self.index]= data.x
        posesy[self.index]= data.y

def update_vel_arrays(v_msgs,velx,vely,index):
    v_msgs[index].linear.x=velx[index]
    v_msgs[index].linear.y=vely[index]

m=[update_callback(i) for i in range(16)]

pubs=[rospy.Publisher('/turtle'+str(i)+'/cmd_vel', Twist, queue_size=10) for i in range(1,17)]
subs=[rospy.Subscriber('/turtle'+str(i)+'/pose', Pose, m[i-1].update_pose) for i in range(1,17)]

v_msgs=[Twist() for i in range(16)]


class BouncyBot:
    def __init__(self):
        self.active_turtles=[1]
        rospy.init_node('BouncyBot2', anonymous=True)
        self.rate = rospy.Rate(50)

    def update_vel(self,turtle_no):
        
        if (posesx[turtle_no]<1) or (posesx[turtle_no]>8):
            if (posesx[turtle_no]>8) and velx[turtle_no]>0:
                    velx[turtle_no]=-1*velx[turtle_no]
            if (posesx[turtle_no]<1) and velx[turtle_no]<0:
                    velx[turtle_no]=-1*velx[turtle_no]
            flag[turtle_no][0]=1

        elif (posesy[turtle_no]<1) or (posesy[turtle_no]>8):
            if (posesy[turtle_no]>8) and vely[turtle_no]>0:
                vely[turtle_no]=-1*vely[turtle_no]
            if (posesy[turtle_no]<1) and vely[turtle_no]<0:
                vely[turtle_no]=-1*vely[turtle_no]
            flag[turtle_no][0]=1
        else:
            flag[turtle_no][0],flag[turtle_no][1]=0,0
            
        if flag[turtle_no][0]==1 and flag[turtle_no][1]==0 and len(self.active_turtles)<16:

            new=len(self.active_turtles)+1
            self.active_turtles.append(new)
            spawn_turtle = rospy.ServiceProxy('spawn', Spawn)
            spawn_turtle(posesx[turtle_no],posesy[turtle_no],0,"turtle"+str(new))
            self.set_vel_newbot(turtle_no,new-1)
            flag[turtle_no][1]=1
            flag[new-1][0],flag[new-1][1]=1,1
        print("Flag",flag)

    # pass before update vel of bot1 to get bot2
    def set_vel_newbot(self,n1,n2):
        print("SPAWNED",len(self.active_turtles),"****************************************************************(n1,n2):",n1+1,n2+1)
        vx=abs(velx[n1])
        vy=abs(vely[n1])
        
        if (posesx[n1]<1):
            if vely[n1]>0:
                velx[n2]=vx
                vely[n2]=-vy
            else:
                velx[n2]=vx
                vely[n2]=vy
        if (posesx[n1]>8):
            if vely[n1]>0:
                velx[n2]= -vx
                vely[n2]= -vy
            else:
                velx[n2]=-vx
                vely[n2]=+vy

        if (posesy[n1]<1):
            if velx[n1]>0:
                velx[n2]=-vx
                vely[n2]=+ vy
            else:
                velx[n2]=+vx
                vely[n2]=+vy
            
        if (posesy[n1]>8):
            if velx[n1]>0:
                velx[n2]= -vx
                vely[n2]= -vy
            else:
                velx[n2]=vx
                vely[n2]=-vy
            
        rospy.sleep(0.1)

        

    def move(self):

        while not rospy.is_shutdown():
            print('posesx:',posesx,'\n','posesy:',posesy)
            print('vel:',velx,'\n','vely:',vely)

            # Keep publishing vel for all active turtles
            for i in range(len(self.active_turtles)):
                
                self.update_vel(i)
                update_vel_arrays(v_msgs,velx,vely,i)
                pubs[i].publish(v_msgs[i])
                self.rate.sleep()
                
        pass



if __name__ == '__main__':
    try:
        x = BouncyBot()
        x.move()
    except rospy.ROSInterruptException:
        pass
