---
title: java实现俄罗斯联机俄罗斯方块
typora-copy-images-to: ../../source/assets
typora-root-url: ../../source
date: 2018-11-17 21:24:34
updated:
tags:
  - socket
categories:
  - java
---

# 简介

本科上java课时做的俄罗斯方块，计算机网络课又把俄罗斯方块改造成了联机对打。房主创建房间，对手通过输入房主的ip来加入对战。其中俄罗斯方块主体部分用到了NetBeans的图形化开发工具，使用idea要装插件。

[github地址](https://github.com/Zlatanlong/JAVA-tetris)

<!-- more -->

# 项目结构

```bash
$***/src tree
.
|____pictures
| |____巴西.png
|____mytetris
| |____CountPanel.java
| |____.DS_Store
| |____CountPanel.form
| |____MainFrame.java
| |____Controller.java
| |____GamePanel.java
| |____GameState.java
| |____GamePanel.form
| |____Block.java
| |____MainFrame.form
|____socket
| |____GameServer.java
| |____SocketUtil.java
| |____GameClient.java
```

其中pictures是俄罗斯方块的一个图片素材，java课时值2018年俄罗斯世界杯，我是🇧🇷球迷，就用了巴西国旗做按钮背景。

`mytetris文件夹`是俄罗斯方块的游戏主体部分，

`socket文件夹`下是俄罗斯方块的联机部分，通过`socket`实现。

# 俄罗斯方块主体代码

## Controller.java

```java
package mytetris;

import java.net.Socket;
import socket.SocketUtil;

public class Controller {

    int removeLog = 0;//同时消行数量
    /**
     * 初始化状态
     */
    int currentX = 3;
    int currentY = 0;
    public int[][] fix = new int[10][20];// 把整个界面分割成10*20
    private GameState state = new GameState();

    /**
     * 产生一个新的形状
     */
    public void getNewShape() {
        getState().getNewBlock();
    }

    public Block getCurrentShape() {
        return getState().getCurrentBlock();
    }

    public Block getNextShape() {
        return getState().getNextBlock();
    }

    /**
     * 初始化界面数组
     */
    public void inintFix() {
        for (int j = 0; j <= 9; j++) {
            for (int k = 0; k <= 19; k++) {
                fix[j][k] = 0;
            }
        }
    }

    /**
     * 判断是否出界
     *
     * @param x X轴偏移量
     * @param y Y轴偏移量
     * @return
     */
    public boolean isValid(int x, int y) {
        int[] tempShape = getCurrentShape().getCurrentBlocks();
        for (int i = 0; i < 8; i += 2) {
            if ((tempShape[i + 1] + y) < 0 || (tempShape[i + 1] + y) > 19) {
                return false;
            }
            if ((tempShape[i] + x) < 0 || (tempShape[i] + x) > 9) {
                return false;
            }
            if (fix[tempShape[i] + x][tempShape[i + 1] + y] != 0) {
                return false;
            }
        }
        return true;
    }

    // 上下左右和翻转 先判断是否出界
    public void left(Boolean ifSend) {
        if (isValid(currentX - 1, currentY)) {
            currentX--;
            if (ifSend) {
                SocketUtil.send(MainFrame.socket, "move37");

            }
        }

    }

    public void right(Boolean ifSend) {
        if (isValid(currentX + 1, currentY)) {
            currentX++;
            if (ifSend) {
                SocketUtil.send(MainFrame.socket, "move39");
            }
        }

    }

    /**
     * 方块下落，落不下去的就死掉了
     */
    public void down(Boolean ifSend) {
        if (ifSend) {
            SocketUtil.send(MainFrame.socket, "move40");
        }
        if (isValid(currentX, currentY + 1)) {
            currentY++;
        } else {
            add(currentX, currentY, ifSend);
        }
    }

    public void turn(Boolean ifSend) {
        getState().getCurrentBlock().next();
        if (!isValid(currentX, currentY)) {
            getState().getCurrentBlock().forward();
        } else {
            if (ifSend) {
                SocketUtil.send(MainFrame.socket, "move38");
            }
        }
    }

    /**
     * 把死掉的方块变成墙；
     *
     * @param x
     * @param y
     */
    public void add(int x, int y, Boolean ifMy) {
        int[] tempShape = getState().getCurrentBlock().getCurrentBlocks();
        for (int i = 0; i < 8; i += 2) {
            fix[x + tempShape[i]][y + tempShape[i + 1]] = getState().getCurrentBlock().getIColor() + 1;
        }
        remove();
        currentX = 3;
        currentY = 0;
        if (ifMy) {
            getNewShape();
            MainFrame.changeNext();
        }
    }

    /**
     * 消除可消的一行
     */
    public void remove() {
        for (int i = 19; i > 0; i--) {
            //i是一共20行
            int flag = 0;
            for (int j = 0; j < 10; j++) {
                if (fix[j][i] == 0) {
                    flag = 1;//一行已经满了
                }
            }
            if (flag == 0) {
                state.setCount(state.getCount() + state.getPoint() + removeLog);
                state.changeInterval();
                MainFrame.changeCount();
                for (int j = 0; j < 10; j++) {
                    fix[j][i] = 0;
                }//消除这一行
                for (int k = i; k > 0; k--) {
                    for (int j = 0; j < 10; j++) {
                        fix[j][k] = fix[j][k - 1];
                    }
                }//其他行下移一行
                removeLog++;
                remove();
            }
        }
        //判断第一行是否要被消除
        int flag0 = 0;
        for (int j = 0; j < 10; j++) {
            if (fix[j][0] == 0) {
                flag0 = 1;
            }
        }
        if (flag0 == 0) {
            for (int j = 0; j < 10; j++) {
                fix[j][0] = 0;
            }
        }
        removeLog = 1;
    }

    /**
     * 道具1
     */
    public void prop1() {
        for (int j = 0; j < 10; j++) {
            fix[j][19] = 0;
        }//消除这一行
        for (int k = 19; k > 0; k--) {
            for (int j = 0; j < 10; j++) {
                fix[j][k] = fix[j][k - 1];
            }
        }//其他行下移一行
        state.setCount(state.getCount() + 10);

        state.changeInterval();
        MainFrame.changeCount();
        MainFrame.gp.repaint();
    }

    /**
     * @return the state
     */
    public GameState getState() {
        return state;
    }

}
```

## Block.java

```java
package mytetris;

import java.awt.Color;
import java.util.List;
import java.util.ArrayList;
import socket.SocketUtil;

/**
 *
 * @author Zlatan
 */
public class Block {
    List<int[][]> allBlocks = new ArrayList<>();
    List<Color> colors = new ArrayList<>();
    int i;
    public Block() {
        allBlocks.add(tblocks);
        colors.add(Color.yellow);
        allBlocks.add(lblocks);
        colors.add(Color.blue);
        allBlocks.add(iblocks);
        colors.add(Color.green);
        allBlocks.add(flblocks);
        colors.add(Color.cyan);
        allBlocks.add(zblocks);
        colors.add(Color.pink);
        allBlocks.add(fzblocks);
        colors.add(Color.white);
        allBlocks.add(oblocks);
        colors.add(Color.orange);
        i=(int)(0+Math.random()*(6-0+1));//(数据类型)(最小值+Math.random()*(最大值-最小值+1))
    }
    
    public Block(int i){
        allBlocks.add(tblocks);
        colors.add(Color.yellow);
        allBlocks.add(lblocks);
        colors.add(Color.blue);
        allBlocks.add(iblocks);
        colors.add(Color.green);
        allBlocks.add(flblocks);
        colors.add(Color.cyan);
        allBlocks.add(zblocks);
        colors.add(Color.pink);
        allBlocks.add(fzblocks);
        colors.add(Color.white);
        allBlocks.add(oblocks);
        colors.add(Color.orange);
        this.i = i;
    }
    
    int state=0;//当前的数
    //格式:(第一块，X轴，Y轴；第二块......)
    //shape of T
    int[][] tblocks={
        {1,0,0,1,1,1,2,1},
        {1,0,1,1,2,1,1,2},
        {1,2,0,1,1,1,2,1},
        {1,0,0,1,1,1,1,2}            
    };
    //shape of L
    int[][] lblocks={
        {1,0,1,1,1,2,2,2},
        {0,1,1,1,2,1,0,2},
        {0,0,1,0,1,1,1,2},
        {2,0,0,1,1,1,2,1}            
    };
    //shape of I
    int[][]iblocks={
        {0,0,1,0,2,0,3,0},
        {2,0,2,1,2,2,2,3},
        {0,0,1,0,2,0,3,0},
        {2,0,2,1,2,2,2,3}
    };
    int[][]flblocks={
        {1,0,1,1,1,2,0,2},
        {0,0,0,1,1,1,2,1},
        {1,0,2,0,1,1,1,2},
        {0,1,1,1,2,1,2,2}   
    };
    int[][]zblocks={
        {1,0,2,0,1,1,0,1},
        {1,0,1,1,2,1,2,2},
        {1,0,2,0,1,1,0,1},
        {1,0,1,1,2,1,2,2}  
    };
    int[][]fzblocks={
        {0,0,1,0,1,1,2,1},
        {1,0,1,1,0,1,0,2},
        {0,0,1,0,1,1,2,1},
        {1,0,1,1,0,1,0,2} 
    };
    int[][]oblocks={
        {1,0,2,0,1,1,2,1},
        {1,0,2,0,1,1,2,1},
        {1,0,2,0,1,1,2,1},
        {1,0,2,0,1,1,2,1}  
    };
    public void next() {
        state = state == 3 ? 0 : state + 1;
    }
    /**
     * 
     */
    public void forward() {
        state = state == 0 ? 3 : state - 1;
    }

    /**
     * 返回当前形状数组
     * @return 
     */
    public int[] getCurrentBlocks() {
            return allBlocks.get(i)[state];
    }
    
    public Color getColor(){
        return colors.get(i);
    }
    
    public int getIColor(){
        if (MainFrame.mod==0) {
            return 8;
        }
        return i;
    }
    
    public int getI() {
        return i;
    }
}
```

## GameState.java

```java
package mytetris;

import socket.SocketUtil;

public class GameState {

    private Block currentBlock; // 当前形状
    private Block nextBlock; // 下一个形状
    private int count; // 总得分
    private int point; // 每次加分
    private int interval; // 时间间隔，影响速度

    public GameState() {
        this.nextBlock = new Block();
        SocketUtil.send(MainFrame.socket, "fBlock" + Integer.toString(nextBlock.getI()));
        this.interval = 1000;
        this.count = 0;
        this.point = 1;
    }

    /**
     * 产生一个新的形状
     */
    public void getNewBlock() {
        currentBlock = nextBlock;
        nextBlock = new Block();
        SocketUtil.send(MainFrame.socket, "nBlock" + Integer.toString(nextBlock.getI()));
    }

    public void setNewBlock(Block block) {
        this.currentBlock = this.nextBlock;
        this.nextBlock = block;
    }

    public void changeInterval() {
        if (MainFrame.mod != 2 && (1000 - count * 9) > 0) {
            interval = 1000 - count * 9;
        }
    }

    /**
     * @return the currentBlock
     */
    public Block getCurrentBlock() {
        return currentBlock;
    }

    /**
     * @param currentBlock the currentBlock to set
     */
    public void setCurrentBlock(Block currentBlock) {
        this.currentBlock = currentBlock;
    }

    /**
     * @return the nextBlock
     */
    public Block getNextBlock() {
        return nextBlock;
    }

    /**
     * @param nextBlock the nextBlock to set
     */
    public void setNextBlock(Block nextBlock) {
        this.nextBlock = nextBlock;
    }

    /**
     * @return the count
     */
    public int getCount() {
        return count;
    }

    /**
     * @param count the count to set
     */
    public void setCount(int count) {
        this.count = count;
    }

    /**
     * @return the point
     */
    public int getPoint() {
        return point;
    }

    /**
     * @param point the point to set
     */
    public void setPoint(int point) {
        this.point = point;
    }

    /**
     * @return the interval
     */
    public int getInterval() {
        return interval;
    }

    /**
     * @param interval the interval to set
     */
    public void setInterval(int interval) {
        this.interval = interval;
    }

}
```

## CountPanel.java

```java
package mytetris;

import java.awt.Color;
import java.awt.Graphics;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 *
 * @author dmt
 */
public class CountPanel extends javax.swing.JPanel implements Runnable {
    private int prop1Count = 5;
    private Block nextBlock;
    public CountPanel() {
        initComponents();
        prop1show.setText("数量："+prop1Count);
    }
    /**
     * 显示结束
     */
    public void showOver() {
        show.setText("GAMEOVER!");
    }
    /**
     * 删除Over标致
     */
    public void delOver() {
        show.setText("");
    }

    @Override
    public void run() {
        while (true) {
            try {
                Thread.currentThread().sleep(1000);
            } catch (InterruptedException ex) {
                Logger.getLogger(GamePanel.class.getName()).log(Level.SEVERE, null, ex);
            }
        }
    }
    
    public void drawBlocks(Graphics g) {
        int[] shape = nextBlock.getCurrentBlocks();
        for (int i = 0; i < shape.length; i += 2) {
            g.setColor(Color.black);
            g.drawRect(20 * (shape[i])+15 , 20 * (shape[i + 1])+250 , 19, 19);
            g.setColor(nextBlock.getColor());
            g.fillRect(20 * (shape[i])+15 , 20 * (shape[i + 1])+250 , 18, 18);
        }
    }
    
    @Override
    public void paintComponent(Graphics g) {
        super.paintComponent(g);
        drawBlocks(g);
    }

    /**
     * This method is called from within the constructor to initialize the form.
     * WARNING: Do NOT modify this code. The content of this method is always
     * regenerated by the Form Editor.
     */
    @SuppressWarnings("unchecked")
    // <editor-fold defaultstate="collapsed" desc="Generated Code">//GEN-BEGIN:initComponents
    private void initComponents() {

        currrentcount = new javax.swing.JLabel();
        count = new javax.swing.JLabel();
        jLabel1 = new javax.swing.JLabel();
        show = new javax.swing.JLabel();
        prop1 = new javax.swing.JButton();
        prop1show = new javax.swing.JLabel();

        currrentcount.setText("当前得分:");

        count.setText("0");

        jLabel1.setText("下一图案：");

        show.setFont(new java.awt.Font("Lucida Grande", 1, 13)); // NOI18N
        show.setForeground(new java.awt.Color(255, 0, 0));

        prop1.setText("道具");
        prop1.setFocusable(false);
        prop1.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                prop1ActionPerformed(evt);
            }
        });

        prop1show.setText("数量：0");

        javax.swing.GroupLayout layout = new javax.swing.GroupLayout(this);
        this.setLayout(layout);
        layout.setHorizontalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(layout.createSequentialGroup()
                .addContainerGap()
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, layout.createSequentialGroup()
                        .addGap(0, 0, Short.MAX_VALUE)
                        .addComponent(count, javax.swing.GroupLayout.PREFERRED_SIZE, 62, javax.swing.GroupLayout.PREFERRED_SIZE))
                    .addComponent(show, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE)
                    .addGroup(layout.createSequentialGroup()
                        .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                            .addComponent(prop1show, javax.swing.GroupLayout.PREFERRED_SIZE, 72, javax.swing.GroupLayout.PREFERRED_SIZE)
                            .addComponent(jLabel1)
                            .addComponent(currrentcount, javax.swing.GroupLayout.PREFERRED_SIZE, 90, javax.swing.GroupLayout.PREFERRED_SIZE)
                            .addComponent(prop1))
                        .addGap(0, 4, Short.MAX_VALUE)))
                .addContainerGap())
        );
        layout.setVerticalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(layout.createSequentialGroup()
                .addGap(31, 31, 31)
                .addComponent(currrentcount, javax.swing.GroupLayout.PREFERRED_SIZE, 33, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addGap(18, 18, 18)
                .addComponent(count, javax.swing.GroupLayout.PREFERRED_SIZE, 52, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                .addComponent(show, javax.swing.GroupLayout.PREFERRED_SIZE, 43, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                .addComponent(jLabel1)
                .addGap(137, 137, 137)
                .addComponent(prop1)
                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.UNRELATED)
                .addComponent(prop1show, javax.swing.GroupLayout.PREFERRED_SIZE, 28, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addContainerGap(199, Short.MAX_VALUE))
        );
    }// </editor-fold>//GEN-END:initComponents

    private void prop1ActionPerformed(java.awt.event.ActionEvent evt) {//GEN-FIRST:event_prop1ActionPerformed
        // 使用道具1:
        if (prop1Count>0&&!MainFrame.isOver) {
//            Controller.prop1();
            prop1Count--;
            prop1show.setText("数量："+prop1Count);
        }
    }//GEN-LAST:event_prop1ActionPerformed

    // Variables declaration - do not modify//GEN-BEGIN:variables
    public javax.swing.JLabel count;
    private javax.swing.JLabel currrentcount;
    private javax.swing.JLabel jLabel1;
    private javax.swing.JButton prop1;
    private javax.swing.JLabel prop1show;
    private javax.swing.JLabel show;
    // End of variables declaration//GEN-END:variables

    public void setProp1Count(int prop1Count) {
        this.prop1Count = prop1Count;
        this.prop1show.setText("数量："+prop1Count);
    }
    
    public void setCountText(int count) {
        this.count.setText(Integer.toString(count));
    }

    /**
     * @param nextBlock the nextBlock to set
     */
    public void setNextBlock(Block nextBlock) {
        this.nextBlock = nextBlock;
    }
}
```

## GamePanel.java

```java
package mytetris;

import java.awt.Color;
import java.awt.Graphics;
import java.net.Socket;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 *
 * @author dmt 整个gamepanel分为10*20（宽*高） 200 块 每一块根据fix[x][y]的值来区分颜色，其中若为0
 * 不单独添加颜色，相当于正常 10为红色，现在导致游戏over的那个形状会被画成红色，其他颜色均代表墙。 1-7代表7中颜色，9为不区分形状颜色时墙的颜色
 */
public class GamePanel extends javax.swing.JPanel implements Runnable {

    Controller controller = new Controller();

    public GamePanel(Boolean ifMy) {
        initComponents();
        controller.getNewShape();
        controller.inintFix();
        if (ifMy) {
            new Thread(this).start();
        }
    }

    @Override
    public void run() {
        while (!MainFrame.isOver) {
            try {
                if (!MainFrame.isPause) {
                    controller.down(true);
                    this.repaint();
                }
                Thread.currentThread().sleep(controller.getState().getInterval());
            } catch (InterruptedException ex) {
                Logger.getLogger(GamePanel.class.getName()).log(Level.SEVERE, null, ex);
            }
        }
    }

    @Override
    public void paintComponent(Graphics g) {
        super.paintComponent(g);
        drawBlocks(g, controller.currentX, controller.currentY);
        drawFix(g);
    }

    /**
     * 画墙
     *
     * @param g
     */
    public void drawFix(Graphics g) {
        for (int j = 0; j <= 9; j++) {
            for (int k = 0; k <= 19; k++) {
                if (controller.fix[j][k] == 1) {
                    g.setColor(Color.black);
                    g.drawRect(20 * j, 20 * k, 19, 19);
                    g.setColor(Color.yellow);
                    g.fillRect(j * 20, k * 20, 18, 18);
                }
                if (controller.fix[j][k] == 2) {
                    g.setColor(Color.black);
                    g.drawRect(20 * j, 20 * k, 19, 19);
                    g.setColor(Color.blue);
                    g.fillRect(j * 20, k * 20, 18, 18);
                }
                if (controller.fix[j][k] == 3) {
                    g.setColor(Color.black);
                    g.drawRect(20 * j, 20 * k, 19, 19);
                    g.setColor(Color.green);
                    g.fillRect(j * 20, k * 20, 18, 18);
                }
                if (controller.fix[j][k] == 4) {
                    g.setColor(Color.black);
                    g.drawRect(20 * j, 20 * k, 19, 19);
                    g.setColor(Color.cyan);
                    g.fillRect(j * 20, k * 20, 18, 18);
                }
                if (controller.fix[j][k] == 5) {
                    g.setColor(Color.black);
                    g.drawRect(20 * j, 20 * k, 19, 19);
                    g.setColor(Color.pink);
                    g.fillRect(j * 20, k * 20, 18, 18);
                }
                if (controller.fix[j][k] == 6) {
                    g.setColor(Color.black);
                    g.drawRect(20 * j, 20 * k, 19, 19);
                    g.setColor(Color.white);
                    g.fillRect(j * 20, k * 20, 18, 18);
                }
                if (controller.fix[j][k] == 7) {
                    g.setColor(Color.black);
                    g.drawRect(20 * j, 20 * k, 19, 19);
                    g.setColor(Color.orange);
                    g.fillRect(j * 20, k * 20, 18, 18);
                }
                if (controller.fix[j][k] == 9) {
                    g.setColor(Color.lightGray);
                    g.fillRect(j * 20, k * 20, 19, 19);
                }
                // 红色警告
                if (controller.fix[j][k] == 10) {
                    g.setColor(Color.red);
                    g.fillRect(j * 20, k * 20, 19, 19);
                }
            }
        }
    }

    /**
     * 画形状 同时判断结束, 判断逻辑： 如果要画形状的时候当前位置已经有墙了，说明墙已经到了最上面， 因此游戏结束
     *
     * @param g
     * @param x
     * @param y
     */
    public void drawBlocks(Graphics g, int x, int y) {
        int[] shape = controller.getCurrentShape().getCurrentBlocks();
//        for (int i = 0; i < shape.length; i++) {
//            int j = shape[i];
//            System.out.print(j);
//        }
//        System.out.println("huanhang");
        for (int i = 0; i < shape.length; i += 2) {
            if (controller.fix[shape[i] + x][shape[i + 1] + y] != 0) {
                // 把结束的这个形状位置号标为10， 会化成红色
                for (int j = 0; j < shape.length; j += 2) {
                    controller.fix[shape[j] + x][shape[j + 1] + y] = 10;
                }
                MainFrame.over();
                return;
            }
        }
        for (int i = 0; i < shape.length; i += 2) {
            g.setColor(Color.black);
            g.drawRect(20 * (shape[i] + x), 20 * (shape[i + 1] + y), 19, 19);
            g.setColor(controller.getCurrentShape().getColor());
            g.fillRect(20 * (shape[i] + x), 20 * (shape[i + 1] + y), 18, 18);
        }
    }

    public Block getNextBlock() {
        return controller.getNextShape();
    }

    /**
     * 展示时设置第一个Block
     *
     * @param block
     */
    public void setFirstBlock(Block block) {
        getGameState().setNextBlock(block);
    }

    public void setNewBlock(Block block) {
        getGameState().setNewBlock(block);
    }

    public GameState getGameState() {
        return controller.getState();
    }

//    public void changeCount() {
//        controller.getState().changeCount();
//    }
    public void keyPressed(java.awt.event.KeyEvent evt) {
        if ((!MainFrame.isOver) && (!MainFrame.isPause)) {
            switch (evt.getKeyCode()) {
                case 37:
                    controller.left(true);
                    break;
                case 39:
                    controller.right(true);
                    break;
                case 40:
                    controller.down(true);
                    break;
                case 38:
                    controller.turn(true);
                    break;
                case 65:
                    controller.left(true);
                    break;
                case 68:
                    controller.right(true);
                    break;
                case 83:
                    controller.down(true);
                    break;
                case 87:
                    controller.turn(true);
                    break;
                default:
                    break;
            }
            repaint();
        }
    }

    public void getPressed(String code) {
        if ((!MainFrame.isOver) && (!MainFrame.isPause)) {
            if (code.equals("move37")) {
                controller.left(false);
            } else if (code.equals("move38")) {
                controller.turn(false);
            } else if (code.equals("move39")) {
                controller.right(false);
            } else if (code.equals("move40")) {
                controller.down(false);
            } else if (code.substring(0, 6).equals("fBlock")) {
                int i = Integer.valueOf(code.substring(6));
                Block block = new Block(i);
                setFirstBlock(block);
            } else if (code.substring(0, 6).equals("nBlock")) {
                int i = Integer.valueOf(code.substring(6));
                Block block = new Block(i);
                setNewBlock(block);
            }
            repaint();
        }
    }

    @SuppressWarnings("unchecked")
    // <editor-fold defaultstate="collapsed" desc="Generated Code">//GEN-BEGIN:initComponents
    private void initComponents() {

        setBackground(new java.awt.Color(204, 255, 204));
        setBorder(javax.swing.BorderFactory.createBevelBorder(javax.swing.border.BevelBorder.LOWERED));

        javax.swing.GroupLayout layout = new javax.swing.GroupLayout(this);
        this.setLayout(layout);
        layout.setHorizontalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGap(0, 396, Short.MAX_VALUE)
        );
        layout.setVerticalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGap(0, 296, Short.MAX_VALUE)
        );
    }// </editor-fold>//GEN-END:initComponents
    // Variables declaration - do not modify//GEN-BEGIN:variables
    // End of variables declaration//GEN-END:variables
}
```

## MainFrame.java

这个类是程序的入口，主界面就是由两组`GamePanel`和`CountPanel`对象组成的，其中gp和cp是自己的画面，gp2和cp2是对手的画面，自己的画面来自于监听键盘的上下左右操作，对手画面来自于对方传来的上下作用动作进行操作。

并且全局变量大多都是放在这个类中，并提供了get方法进行修改。

```java
package mytetris;

import java.net.Socket;
import java.util.logging.Level;
import java.util.logging.Logger;
import socket.GameClient;
import socket.GameServer;

public class MainFrame extends javax.swing.JFrame {

    static Socket socket;
    static int mod;
    static boolean isOver = true;
    static boolean isPause = false;
    static int point;//每次加分
    static GamePanel gp;
    static GamePanel gp2;
    static CountPanel cp;
    static CountPanel cp2;
    GameServer server;

    /**
     * 改变分数显示同时改变速度
     */
    static public void changeCount() {
        cp.setCountText(gp.getGameState().getCount());
        cp2.setCountText(gp2.getGameState().getCount());
    }

    static public void over() {
        isOver = true;
        isPause = false;
        cp.showOver();
        cp2.showOver();
    }

    static public void changeNext() {
        cp.setNextBlock(gp.getNextBlock());
        cp.repaint();
        try {
            Thread.sleep(100);
        } catch (InterruptedException ex) {
            Logger.getLogger(MainFrame.class.getName()).log(Level.SEVERE, null, ex);
        }
        cp2.setNextBlock(gp2.getNextBlock());
        cp2.repaint();
        changeCount();
    }

    public MainFrame() {
        //规定画板大小
        initComponents();
        start.setVisible(false);
        pause.setVisible(false);

        cp = new CountPanel();
        cp.setSize(100, 600);
        cp.setLocation(254, 100);
        cp.setVisible(false);
        this.getContentPane().add(cp);

        cp2 = new CountPanel();
        cp2.setSize(100, 600);
        cp2.setLocation(654, 100);
        cp2.setVisible(false);
        this.getContentPane().add(cp2);

        this.setSize(800, 600);
    }

    @SuppressWarnings("unchecked")
    // <editor-fold defaultstate="collapsed" desc="Generated Code">//GEN-BEGIN:initComponents
    private void initComponents() {

        go = new javax.swing.JButton();
        start = new javax.swing.JButton();
        pause = new javax.swing.JButton();
        msg = new javax.swing.JLabel();
        jLabel1 = new javax.swing.JLabel();
        model = new javax.swing.JComboBox<>();
        jScrollPane1 = new javax.swing.JScrollPane();
        msg2 = new javax.swing.JTextArea();
        openHomeButton = new javax.swing.JButton();
        joinHomeButton = new javax.swing.JButton();
        joinHomeText = new javax.swing.JTextField();

        setDefaultCloseOperation(javax.swing.WindowConstants.EXIT_ON_CLOSE);
        setIconImages(null);
        addKeyListener(new java.awt.event.KeyAdapter() {
            public void keyPressed(java.awt.event.KeyEvent evt) {
                formKeyPressed(evt);
            }
        });

        go.setBackground(new java.awt.Color(255, 255, 204));
        go.setFont(new java.awt.Font("Lucida Grande", 1, 64)); // NOI18N
        go.setForeground(new java.awt.Color(102, 255, 102));
        go.setIcon(new javax.swing.ImageIcon(getClass().getResource("/pictures/巴西.png"))); // NOI18N
        go.setToolTipText("");
        go.setBorder(new javax.swing.border.SoftBevelBorder(javax.swing.border.BevelBorder.RAISED));
        go.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                goActionPerformed(evt);
            }
        });

        start.setText("重新开始");
        start.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                startActionPerformed(evt);
            }
        });

        pause.setText("暂停");
        pause.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                pauseActionPerformed(evt);
            }
        });

        msg.setText("双人游戏哦！");

        jLabel1.setText("模式选择：");

        model.setModel(new javax.swing.DefaultComboBoxModel<>(new String[] { "简约模式", "炫彩模式", "极速模式" }));
        model.setFocusable(false);
        model.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                modelActionPerformed(evt);
            }
        });

        msg2.setColumns(20);
        msg2.setRows(5);
        msg2.setText("游戏说明：\n键盘←→控制方向，↑旋转俄罗斯方块，↓加速下落\n简约模式，炫彩模式每消除一行加1分；\n极速模式每消除一行加10分；\n道具说明：\n使用道具后将在最低端消除一行，\n每局游戏只有5次使用道具的机会！");
        msg2.setBorder(null);
        msg2.setFocusable(false);
        jScrollPane1.setViewportView(msg2);

        openHomeButton.setText("创建房间");
        openHomeButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                openHomeButtonActionPerformed(evt);
            }
        });

        joinHomeButton.setText("加入房间");
        joinHomeButton.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                joinHomeButtonActionPerformed(evt);
            }
        });

        joinHomeText.addActionListener(new java.awt.event.ActionListener() {
            public void actionPerformed(java.awt.event.ActionEvent evt) {
                joinHomeTextActionPerformed(evt);
            }
        });

        javax.swing.GroupLayout layout = new javax.swing.GroupLayout(getContentPane());
        getContentPane().setLayout(layout);
        layout.setHorizontalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(layout.createSequentialGroup()
                .addGap(35, 35, 35)
                .addComponent(start, javax.swing.GroupLayout.PREFERRED_SIZE, 106, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addGap(18, 18, 18)
                .addComponent(pause)
                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED, 426, Short.MAX_VALUE)
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addComponent(jLabel1, javax.swing.GroupLayout.PREFERRED_SIZE, 76, javax.swing.GroupLayout.PREFERRED_SIZE)
                    .addComponent(model, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE))
                .addGap(32, 32, 32))
            .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, layout.createSequentialGroup()
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addGroup(layout.createSequentialGroup()
                        .addGap(50, 50, 50)
                        .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, 293, javax.swing.GroupLayout.PREFERRED_SIZE))
                    .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, layout.createSequentialGroup()
                        .addContainerGap()
                        .addComponent(go, javax.swing.GroupLayout.PREFERRED_SIZE, 293, javax.swing.GroupLayout.PREFERRED_SIZE)))
                .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.UNRELATED)
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addGroup(layout.createSequentialGroup()
                        .addComponent(openHomeButton, javax.swing.GroupLayout.PREFERRED_SIZE, 224, javax.swing.GroupLayout.PREFERRED_SIZE)
                        .addGap(0, 0, Short.MAX_VALUE))
                    .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, layout.createSequentialGroup()
                        .addGap(0, 16, Short.MAX_VALUE)
                        .addComponent(joinHomeButton, javax.swing.GroupLayout.PREFERRED_SIZE, 152, javax.swing.GroupLayout.PREFERRED_SIZE)
                        .addGap(18, 18, 18)
                        .addComponent(joinHomeText, javax.swing.GroupLayout.PREFERRED_SIZE, 153, javax.swing.GroupLayout.PREFERRED_SIZE)
                        .addGap(106, 106, 106))))
            .addGroup(javax.swing.GroupLayout.Alignment.TRAILING, layout.createSequentialGroup()
                .addContainerGap(javax.swing.GroupLayout.DEFAULT_SIZE, Short.MAX_VALUE)
                .addComponent(msg, javax.swing.GroupLayout.PREFERRED_SIZE, 293, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addGap(199, 199, 199))
        );
        layout.setVerticalGroup(
            layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
            .addGroup(layout.createSequentialGroup()
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING)
                    .addGroup(layout.createSequentialGroup()
                        .addGap(25, 25, 25)
                        .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.BASELINE)
                            .addComponent(start, javax.swing.GroupLayout.PREFERRED_SIZE, 50, javax.swing.GroupLayout.PREFERRED_SIZE)
                            .addComponent(pause, javax.swing.GroupLayout.PREFERRED_SIZE, 50, javax.swing.GroupLayout.PREFERRED_SIZE)))
                    .addGroup(layout.createSequentialGroup()
                        .addGap(12, 12, 12)
                        .addComponent(jLabel1, javax.swing.GroupLayout.PREFERRED_SIZE, 26, javax.swing.GroupLayout.PREFERRED_SIZE)
                        .addPreferredGap(javax.swing.LayoutStyle.ComponentPlacement.RELATED)
                        .addComponent(model, javax.swing.GroupLayout.PREFERRED_SIZE, javax.swing.GroupLayout.DEFAULT_SIZE, javax.swing.GroupLayout.PREFERRED_SIZE)))
                .addGap(20, 20, 20)
                .addComponent(msg, javax.swing.GroupLayout.PREFERRED_SIZE, 40, javax.swing.GroupLayout.PREFERRED_SIZE)
                .addGap(68, 68, 68)
                .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.TRAILING)
                    .addGroup(javax.swing.GroupLayout.Alignment.LEADING, layout.createSequentialGroup()
                        .addComponent(go, javax.swing.GroupLayout.PREFERRED_SIZE, 198, javax.swing.GroupLayout.PREFERRED_SIZE)
                        .addGap(18, 18, 18)
                        .addComponent(jScrollPane1, javax.swing.GroupLayout.PREFERRED_SIZE, 120, javax.swing.GroupLayout.PREFERRED_SIZE))
                    .addGroup(javax.swing.GroupLayout.Alignment.LEADING, layout.createSequentialGroup()
                        .addComponent(openHomeButton, javax.swing.GroupLayout.PREFERRED_SIZE, 142, javax.swing.GroupLayout.PREFERRED_SIZE)
                        .addGap(44, 44, 44)
                        .addGroup(layout.createParallelGroup(javax.swing.GroupLayout.Alignment.LEADING, false)
                            .addComponent(joinHomeButton, javax.swing.GroupLayout.DEFAULT_SIZE, 173, Short.MAX_VALUE)
                            .addComponent(joinHomeText))))
                .addContainerGap(106, Short.MAX_VALUE))
        );

        pack();
    }// </editor-fold>//GEN-END:initComponents

    private void formKeyPressed(java.awt.event.KeyEvent evt) {//GEN-FIRST:event_formKeyPressed
        try {
            if (evt.getKeyCode() < 50) {
                gp.keyPressed(evt);
            } else {
            }
        } catch (NullPointerException e) {
            System.out.println("还没点开始");
        }
    }//GEN-LAST:event_formKeyPressed

    private void goActionPerformed(java.awt.event.ActionEvent evt) {//GEN-FIRST:event_goActionPerformed
        // 开始游戏按钮:
//        MainFrame.isOver = false;
//        msg.setVisible(false);
//        msg2.setVisible(false);
//        jScrollPane1.setVisible(false);
//        start.setVisible(true);
//        pause.setVisible(true);
//        gp2 = new GamePanel(false);
//        gp2.setSize(200, 400);
//        gp2.setLocation(450, 100);
//        this.getContentPane().add(gp2);
//
//        server = new GameServer();
//        ClientListen clientListen = new ClientListen(gp2);
//        setSocket(server.getSocket());
//        
//        gp = new GamePanel(true);
//        gp.setSize(200, 400);
//        gp.setLocation(50, 100);
//        this.getContentPane().add(gp);
//
//        changeNext();
//
//        cp.setVisible(true);
//        cp2.setVisible(true);
//
//        go.setVisible(false);
//        this.requestFocusInWindow();
    }//GEN-LAST:event_goActionPerformed

    private void pauseActionPerformed(java.awt.event.ActionEvent evt) {//GEN-FIRST:event_pauseActionPerformed
        // 暂停按钮:
        if (isPause) {
            isPause = false;
            pause.setText("暂停");
        } else {
            isPause = true;
            pause.setText("继续");
        }
        this.requestFocusInWindow();
    }//GEN-LAST:event_pauseActionPerformed

    private void startActionPerformed(java.awt.event.ActionEvent evt) {//GEN-FIRST:event_startActionPerformed
        // 重新开始按钮:
        try {
            over();
            Thread.sleep(1000);//等待GamePanel线程死亡
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        MainFrame.isOver = false;
        gp = new GamePanel(true);
        gp.setSize(200, 400);
        gp.setLocation(50, 100);
        this.getContentPane().add(gp);

        gp2 = new GamePanel(false);
        gp2.setSize(200, 400);
        gp2.setLocation(450, 100);
        this.getContentPane().add(gp2);

        changeCount();
        changeNext();
        cp.delOver();
        cp.setProp1Count(5);
        cp2.delOver();
        cp2.setProp1Count(5);
        this.requestFocusInWindow();
    }//GEN-LAST:event_startActionPerformed

    private void modelActionPerformed(java.awt.event.ActionEvent evt) {//GEN-FIRST:event_modelActionPerformed
        // 模式切换
        String mo = (String) model.getSelectedItem();
        switch (mo) {
            case "简约模式":
                mod = 0;
                point = 1;
                break;
            case "炫彩模式":
                mod = 1;
                point = 1;
                break;
            case "极速模式":
                mod = 2;
                gp.getGameState().setInterval(100);
                gp2.getGameState().setInterval(100);
                point = 10;
                break;
            default:
                break;
        }
    }//GEN-LAST:event_modelActionPerformed

    private void openHomeButtonActionPerformed(java.awt.event.ActionEvent evt) {//GEN-FIRST:event_openHomeButtonActionPerformed
        // 开房
        MainFrame.isOver = false;
        msg.setVisible(false);
        msg2.setVisible(false);
        jScrollPane1.setVisible(false);
        start.setVisible(true);
        pause.setVisible(true);
        gp2 = new GamePanel(false);
        gp2.setSize(200, 400);
        gp2.setLocation(450, 100);
        this.getContentPane().add(gp2);

        server = new GameServer(gp2);
//        ClientListen clientListen = new ClientListen(gp2);
        setSocket(server.getSocket());
        
        gp = new GamePanel(true);
        gp.setSize(200, 400);
        gp.setLocation(50, 100);
        this.getContentPane().add(gp);

        changeNext();

        cp.setVisible(true);
        cp2.setVisible(true);

        go.setVisible(false);
        openHomeButton.setVisible(false);
        joinHomeButton.setVisible(false);
        joinHomeText.setVisible(false);
        this.requestFocusInWindow();
    }//GEN-LAST:event_openHomeButtonActionPerformed

    private void joinHomeTextActionPerformed(java.awt.event.ActionEvent evt) {//GEN-FIRST:event_joinHomeTextActionPerformed
        // TODO add your handling code here:
    }//GEN-LAST:event_joinHomeTextActionPerformed

    private void joinHomeButtonActionPerformed(java.awt.event.ActionEvent evt) {//GEN-FIRST:event_joinHomeButtonActionPerformed
        // 加入房间
        MainFrame.isOver = false;
        msg.setVisible(false);
        msg2.setVisible(false);
        jScrollPane1.setVisible(false);
        start.setVisible(true);
        pause.setVisible(true);
        gp2 = new GamePanel(false);
        gp2.setSize(200, 400);
        gp2.setLocation(450, 100);
        this.getContentPane().add(gp2);

        GameClient clientListen = new GameClient(gp2, joinHomeText.getText());
        setSocket(clientListen.getSocket());
        
        gp = new GamePanel(true);
        gp.setSize(200, 400);
        gp.setLocation(50, 100);
        this.getContentPane().add(gp);

        changeNext();

        cp.setVisible(true);
        cp2.setVisible(true);

        go.setVisible(false);
        openHomeButton.setVisible(false);
        joinHomeButton.setVisible(false);
        joinHomeText.setVisible(false);
        this.requestFocusInWindow();
    }//GEN-LAST:event_joinHomeButtonActionPerformed

    /**
     * @param args the command line arguments
     */
    public static void main(String args[]) {
        /* Set the Nimbus look and feel */
        //<editor-fold defaultstate="collapsed" desc=" Look and feel setting code (optional) ">
        /* If Nimbus (introduced in Java SE 6) is not available, stay with the default look and feel.
         * For details see http://download.oracle.com/javase/tutorial/uiswing/lookandfeel/plaf.html 
         */
        try {
            for (javax.swing.UIManager.LookAndFeelInfo info : javax.swing.UIManager.getInstalledLookAndFeels()) {
                if ("Nimbus".equals(info.getName())) {
                    javax.swing.UIManager.setLookAndFeel(info.getClassName());
                    break;
                }
            }
        } catch (ClassNotFoundException ex) {
            java.util.logging.Logger.getLogger(MainFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        } catch (InstantiationException ex) {
            java.util.logging.Logger.getLogger(MainFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        } catch (IllegalAccessException ex) {
            java.util.logging.Logger.getLogger(MainFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        } catch (javax.swing.UnsupportedLookAndFeelException ex) {
            java.util.logging.Logger.getLogger(MainFrame.class.getName()).log(java.util.logging.Level.SEVERE, null, ex);
        }
        //</editor-fold>

        /* Create and display the form */
        java.awt.EventQueue.invokeLater(new Runnable() {
            public void run() {
                MainFrame mf = new MainFrame();
                mf.setVisible(true);
                mf.requestFocusInWindow();
            }
        });
    }
    // Variables declaration - do not modify//GEN-BEGIN:variables
    private javax.swing.JButton go;
    private javax.swing.JLabel jLabel1;
    private javax.swing.JScrollPane jScrollPane1;
    private javax.swing.JButton joinHomeButton;
    private javax.swing.JTextField joinHomeText;
    private javax.swing.JComboBox<String> model;
    private javax.swing.JLabel msg;
    private javax.swing.JTextArea msg2;
    private javax.swing.JButton openHomeButton;
    private javax.swing.JButton pause;
    private javax.swing.JButton start;
    // End of variables declaration//GEN-END:variables
    public static Socket getSocket() {
        return socket;
    }

    public static void setSocket(Socket s) {
        socket = s;
    }
}
```



# socket实现联机

这里实现联机有很多思路，比如传显示对方的结果的矩阵，或者只传递动作。这里我选择了只传递动作参数，在对战双方的程序中各自进行运算。

## SocketUnit.java

```java
package socket;

import java.io.IOException;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.Socket;
import java.util.logging.Level;
import java.util.logging.Logger;

public class SocketUtil {
    
    public static void send(Socket socket,String string) {
        if (socket != null) {
            try {
                PrintWriter pw = new PrintWriter(
                        new OutputStreamWriter(
                                socket.getOutputStream()));
                pw.println(string);
                pw.flush();
            } catch (IOException ex) {
                System.out.println("error!!");
            }
        }
    }
}
```

## GameServer.java

```java
package socket;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.logging.Level;
import java.util.logging.Logger;
import mytetris.GamePanel;

public class GameServer extends Thread {

    Socket socket;
    GamePanel gamePanel;

    
    public GameServer(GamePanel gamePanel) {
        try {
            this.gamePanel = gamePanel;
            System.out.println("服务器开启！");
            ServerSocket serverSocket = new ServerSocket(50000);
            socket = serverSocket.accept();
            System.out.println("----------------------------");
            System.out.println("有新的客户端连接到服务器喽！");
        new Thread(this).start();
        } catch (IOException ex) {
            Logger.getLogger(GameServer.class.getName()).log(Level.SEVERE, null, ex);
        }
    }

    @Override
    public void run() {
        System.out.println("我开始收听客户端！");
        try {
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(
                            socket.getInputStream()));
            String code;
            while (!(code = br.readLine()).equals("bye")) {
                gamePanel.getPressed(code);
                gamePanel.repaint();
            }
            System.out.println("客户端已经关闭了对话！");
            br.close();
            socket.close();
        } catch (Exception e) {
            System.out.println("服务器已经关闭");
            e.printStackTrace();
        }
    }

    public Socket getSocket() {
        return socket;
    }
}
```

## GameClient.java

```java
package socket;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.Socket;
import java.util.logging.Level;
import java.util.logging.Logger;
import mytetris.GamePanel;

public class GameClient extends Thread {

    Socket clientSocket;
    GamePanel gamePanel;

    public GameClient(GamePanel gamePanel, String ip) {
        try {
            this.gamePanel = gamePanel;
            this.clientSocket = new Socket(ip, 50000);
            new Thread(this).start();
        } catch (IOException ex) {
            Logger.getLogger(GameClient.class.getName()).log(Level.SEVERE, null, ex);
        }
    }

    @Override
    public void run() {
        try {
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(
                            clientSocket.getInputStream()));
            String code;
            while (!(code = br.readLine()).equals("bye")) {
                gamePanel.getPressed(code);
                gamePanel.repaint();
            }
            System.out.println("服务端已经关闭了对话！");
            br.close();
            clientSocket.close();
        } catch (Exception e) {
            System.out.println("客户端已经关闭");
            e.printStackTrace();
        }
    }
    
    public Socket getSocket() {
        return clientSocket;
    }
}
```

