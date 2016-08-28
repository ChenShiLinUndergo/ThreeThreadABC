# ThreeThreadABCpackage multipleThread;

/**
 * 三线程交替打印ABC
 * Created by shilin on 2016/8/21.
 */
public class ThreeThreadABC {
    public class Printer implements Runnable{
        private String pre; //注意不要进入误区，这里的参数不能定义成静态的，
                         // 如果定义成静态的把值设置进来了之后就变成这个类共享的了，
                             // 然后三个对象（线程）实例化后都去更改它，那就完蛋了，
        private String self;// 就不知道这个到底是干啥的了，不能把变量当成共享的，
                         // main方法里面那三个String才是共享的，每次申请的锁来自于它们
        public Printer(String pre, String self) {
            this.pre = pre;
            this.self = self;
        }
        @Override
        public void run() {
            for (int i = 0 ; i < 10; i ++) {
                synchronized (pre) {
                    synchronized (self) {
                        System.out.print(self+" ");
                        sleep(100);
                        self.notify();
                        //为什么这里要用notify下面为什么要用wait？
                        // 不要被字面上的理解所蒙蔽，
                        //首先要知道它们各自的功能，
                        //notify会唤醒其他线程，释放self的锁供下个线程获取，
                        // 而为什么下面那个要用wait呢?
                        // 是因为你如果直接用notify，那好，这个线程算执行完了，
                        // 不会再回来执行for循环剩下的东西了，
                        // 而用wait是释放锁，自己休息去了，
                        // 然后等待别的线程执行完释放这个的锁后自己还可以再执行,
                        // 也就是线程会等在这里（for循环的某次结束）
                        // 然后等待下一次得到这个锁的时候，从这个结束重新进入新的循环
                    }
                    try {
                        pre.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
    public static void sleep(long time){
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args){
        ThreeThreadABC t = new ThreeThreadABC();
        String A = "A";     //三个共享变量
        String B = "B";
        String C = "C";
        Printer P1 = t.new Printer(C,A);
        Printer P2 = t.new Printer(A,B);
        Printer P3 = t.new Printer(B,C);
        new Thread(P1).start();
        sleep(100);         //这里是为了保证按顺序启动进程
        new Thread(P2).start();
        sleep(100);
        new Thread(P3).start();
        sleep(100);
    }
}
