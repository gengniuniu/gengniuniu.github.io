---
layout: post
title: 单一职责原则java
---

登录模块在实际项目开发中很常见，请按照教材28页利用单一职责原则重构后的类图实现这一模块。



 

 



 

 

 代码按照图中顺序

复制代码
复制代码
package test1;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
public class DBUtil {

    public static String db_url = "jdbc:mysql://localhost:3306/first?serverTimezone=GMT%2B8";
    public static String db_user = "root";
    public static String db_pass = "123456";

    public static Connection getConn () {
        Connection conn = null;

        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection(db_url, db_user, db_pass);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return conn;
    }//end getConn

    public static void close (Statement state, Connection conn) {
        if (state != null) {
            try {
                state.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    public static void close (ResultSet rs, Statement state, Connection conn) {
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (state != null) {
            try {
                state.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws SQLException {
        Connection conn = getConn();
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        String sql ="select * from user";
        pstmt = conn.prepareStatement(sql);
        rs = pstmt.executeQuery();
        if(rs.next()){
            System.out.println("连接成功");
        }else{
            System.out.println("连接失败");
        }
    }
}
复制代码
复制代码
复制代码
复制代码
package test1;


import java.awt.*;
import java.awt.event.*;
import javax.swing.*;

public class LoginForm extends JFrame {

    private static final long serialVersionUID = 1L;
    private  UserDao dao = new UserDao();
    //设置按钮组件

    private  JButton jb=new JButton("登录");    //添加一个登录按钮
    private JButton button=new JButton("重置");    //添加一个确定按钮
    //设置文本框组件
    private JTextField username = new JTextField();//用户名框
    private JPasswordField password = new JPasswordField();//密码框：为加密的***

    JLabel user_name=new JLabel("账号：");//设置左侧用户名文字
    JLabel pass_word=new JLabel("密码：");//设置左侧密码文字

    public void init()
    {
        /* 组件绝对位置  */
        user_name.setBounds(50, 70, 300, 25);
        pass_word.setBounds(50, 130, 200, 25);

        username.setBounds(110, 70, 300, 25);//设置用户名框的宽，高，x值，y值
        password.setBounds(110, 130, 300, 25);//设置密码框的宽，高，x值，y值

        button.setBounds(315, 225, 90, 20);//设置确定按钮的宽，高，x值，y值
        jb.setBounds(95, 225, 90, 20);//设置确定按钮的宽，高，x值，y值


        /* 组件透明化*/
        user_name.setOpaque(false);
        pass_word.setOpaque(false);


        //监听事件
        jb.addActionListener(new ActionListener(){        //为确定按钮添加监听事件

            @SuppressWarnings("deprecation")
            public void actionPerformed(ActionEvent arg0) {
                validate(username.getText().trim(),password.getText().trim());
            }
        });


        //重置按钮
        button.addActionListener(new ActionListener(){        //为重置按钮添加监听事件
            //同时清空name、password的数据
            public void actionPerformed(ActionEvent arg0) {
                // TODO 自动生成方法存根
                username.setText("");
                password.setText("");
            }
        });

    }

    public void display()
    {
        JFrame f =new JFrame();
        f.setTitle("登录页面");
        //窗口退出行为
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        //设置窗口大小不可变
        f.setResizable(false);
        //设置窗口打开居中
        f.setLocationRelativeTo(null);
        //窗口大小
        f.setSize(500, 300);

        init();

        //添加组件
        Container contentPanel= new Container();//添加一个contentPanel容器
        contentPanel.setLayout(null);//设置添加的contentPanel容器为流布局管理器
        contentPanel.add(user_name);
        contentPanel.add(pass_word);
        contentPanel.add(username);
        contentPanel.add(password);
        contentPanel.add(jb);
        contentPanel.add(button);

        f.add(contentPanel);
        //展示窗口
        f.setVisible(true);
    }


    public  void  validate(String username,String password)
    {

        if(username.trim().length()==0||password.trim().length()==0){
            JOptionPane.showMessageDialog(null, "用户名,密码不允许为空");

            return;
        }

        if(dao.findUser(username, password))
        {

            JOptionPane.showMessageDialog(null, "登录成功！");


        }else {
            JOptionPane.showMessageDialog(null, "用户名或密码错误");

        }

    }

}
复制代码
复制代码
复制代码
复制代码
package test1;



public class MainClass {

    public static void main(String[] args)
    {
        LoginForm loginForm=new LoginForm() ;    //调用
        loginForm.display();

    }

}


//run此文件
复制代码
复制代码
复制代码
复制代码
package test1;


import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class UserDao {
    public boolean findUser(String username, String password) {
        //准备SQL语句
        String sql = "select * from user where username ='" + username + "'";
        Connection conn= DBUtil.getConn();
        //创建语句传输对象
        Statement state = null;
        ResultSet rs= null;
        int flag=0;
        String c_password=null;
        try {
            state = conn.createStatement();
            rs = state.executeQuery(sql);
            while(rs.next()) {
                ++flag;
                c_password=rs.getString("password");
            }    if (flag == 0) {
                return false;
            }
            if (!password.equals(c_password)) {   //判断密码
                return false;
            }
        }catch (SQLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }finally {
            DBUtil.close(rs, state, conn);
        }
        return true;
    }
}
复制代码
复制代码
 

sql

复制代码
复制代码
CREATE TABLE `user`  (
  `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('张三', '123456');

SET FOREIGN_KEY_CHECKS = 1;
复制代码
复制代码
 

 

 

 

 



 

 

 
