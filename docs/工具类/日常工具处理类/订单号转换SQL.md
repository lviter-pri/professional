# 订单号转换SQL

## 订单号转换工具

1. 一批订单号，转换为id，根据id生成update语句

   ```java
   package com.omega.biz.example.provider.util;
   
   import java.io.*;
   import java.nio.file.Files;
   import java.nio.file.Path;
   import java.nio.file.Paths;
   import java.util.ArrayList;
   import java.util.List;
   import java.util.Scanner;
   
   /**
    * 快速工具
    *
    * @author Lv.
    * @date 2026/3/25 15:07
    */
   public class QuickTool {
       public static void main(String[] args) throws Exception {
           convertIn();
   //        convert();
   //        convertBatch();
       }
   
       public static void convertIn() throws FileNotFoundException {
           Scanner sc = new Scanner(new File("D:\\refresh.txt"));
           List<String> list = new ArrayList<>();
   
           while (sc.hasNext()) {
               String token = sc.next();
               // 简单逻辑：如果这一段内容全是数字且长度符合要求
               list.add(token);
           }
           sc.close();
           //将list内的每个字符串加一个单引号，然后转成字符串，逗号分隔
           StringBuilder sb = new StringBuilder();
           for (String s : list) {
               sb.append("'").append(s).append("',");
           }
           //去除最后一个逗号
           sb.deleteCharAt(sb.length() - 1);
           System.out.println("select id\n" +
                   "from finance.payable_bill_detail\n" +
                   "where trade_number in (" + sb + ");");
       }
   
   
       /**
        * 一批订单id转换为更新sql语句
        *
        * @throws IOException
        */
       public static void convert() throws IOException {
           // 1. 读取（沿用你的Scanner逻辑，确保在你的环境下不乱码）
           File inputFile = new File("D:\\refresh_01.txt");
           if (!inputFile.exists()) return;
   
           Scanner sc = new Scanner(inputFile);
           List<String> list = new ArrayList<>();
           while (sc.hasNext()) {
               list.add(sc.next());
           }
           sc.close();
   
           // 2. 写入文件（避免控制台显示不全）
           Path outputPath = Paths.get("D:\\update_orders.txt");
           try (BufferedWriter writer = Files.newBufferedWriter(outputPath)) {
               for (String id : list) {
                   String sql = String.format(
                           "UPDATE finance.payable_bill_detail SET tms_unpaid_amount = 0, tms_actual_paid_amount = NULL, tms_paid_amount = NULL, tms_allocation_amount = NULL WHERE id = %s;\n",
                           id
                   );
                   writer.write(sql);
               }
           }
           System.out.println("处理完成！SQL文件已生成至: " + outputPath.toAbsolutePath());
           System.out.println("总记录数: " + list.size());
       }
   
       /**
        * 一批订单id转换为批量更新sql语句
        *
        * @throws IOException
        */
       public static void convertBatch() throws IOException {
           List<String> ids = new ArrayList<>();
           try (Scanner sc = new Scanner(new File("D:\\refresh_01.txt"))) {
               while (sc.hasNext()) ids.add(sc.next());
           }
   
           // 每500个ID拼成一条SQL
           int batchSize = 500;
           try (PrintWriter pw = new PrintWriter("D:\\batch_update.sql")) {
               for (int i = 0; i < ids.size(); i += batchSize) {
                   int end = Math.min(i + batchSize, ids.size());
                   List<String> subList = ids.subList(i, end);
   
                   // 拼接 ID 列表，如: 'ID1','ID2'...
                   String idRange = String.join("','", subList);
   
                   pw.println("UPDATE finance.payable_bill_detail ");
                   pw.println("SET tms_unpaid_amount = 0, tms_actual_paid_amount = NULL, tms_paid_amount = NULL, tms_allocation_amount = NULL ");
                   pw.println("WHERE id IN ('" + idRange + "');");
                   pw.println(); // 换行增强可读性
               }
           }
           System.out.println("批量SQL已生成，大幅提升执行速度。");
       }
   }
   
   ```

   

2. 1