~~~
package work.easyWork;

import com.alibaba.fastjson.JSONObject;
import org.apache.commons.lang3.StringUtils;
import work.constant.AuthorInfoEnum;
import work.constant.MemberNameEnum;
import work.utils.SHA256Util;

import java.io.*;
import java.util.*;
import java.util.regex.Pattern;

/**
 * @author yangyunchao
 * @date 2020/9/27
 */
public class Operation {

    // 换行
    private static final String SEPARATOR = "line.separator";

//    private static String PATH = "D:\\ProjectSpace\\conflict\\EUMS\\";
//    private static String PATH = "D:\\ProjectSpace\\IDEASpace\\EUMS\\";
    private static String PATH = "D:\\ProjectSpace\\conflict\\UDMS\\";
//    private static String PATH = "D:\\ProjectSpace\\IDEASpace\\UDMS\\";

    public static void main(String[] args) throws IOException, ClassNotFoundException {
//        addAnnoation();
//        createAccount();

        Properties properties = loadProperties();
        for (Map.Entry<Object, Object> entry : properties.entrySet()) {
            System.out.println(entry.getValue());
            modifyMemberNameAsHump(PATH + entry.getValue());
//            deleteAuthor(PATH + entry.getValue());
        }
    }

    /**
     * 修改变量名为驼峰
     *
     * @param path
     * @throws IOException
     */
    public static void modifyMemberNameAsHump(String path) throws IOException, ClassNotFoundException {
        File file = new File(path);
        if (!file.exists()) {
            return;
        }
        File[] fs = file.listFiles();
        for (File f : fs) {
            if (f.isDirectory()) {
                // 限定包
                if (!MemberNameEnum.isPackageInstance(f.getName())) {
                    continue;
                }
                modifyMemberNameAsHump(f.getPath());
                continue;
            }
            FileReader fileReader = new FileReader(f);
            BufferedReader bufferedReader = new BufferedReader(fileReader);//把文件存到缓冲区
            StringBuilder sb = new StringBuilder();
            // 只修改 java 文件
            if (!f.getName().endsWith("java")) {
                continue;
            }

            /*
            // ===== 方法一：注解 === 拉闸了拉闸了
            if (f.getName().contains("Dto") || f.getName().contains("Vo")) {
                boolean isRightName = false;
                Class clazz = Class.forName(f.getName());
                Field[] declaredFields = clazz.getDeclaredFields();
                for (Field field : declaredFields) {
                    // 是否有 JsonProperty 注解
                    if (field.isAnnotationPresent(JsonProperty.class)) {
                        isRightName = true;
                    }
                    if (!isRightName && Modifier.isPrivate(field.getModifiers()) && field.getName().contains("_")) {
                        field.setAccessible(true);
                        // TODO  呜呜呜 反射做不到 给属性增加注解 和 修改属性名
                        isRightName = false;
                    }
                }
            }
            */

            // ===== 方法二：文件操作 字符串暴力更换 ===
            // 入参实体类
//            if (f.getName().contains("Dto")) {
                System.out.println(f.getName());
                // 修改的行数
                int line = 0;
                String str = bufferedReader.readLine();
                // 存在 jsonProperty ,直接记录下来，下一行重新加上
                String jsonProperty = null;
                while (Objects.nonNull(str)) {
                    line++;
                    if (line == 3
//                            && (f.getName().contains("Dto") || f.getName().contains("Vo"))
                    ) {
                        sb.append("import com.fasterxml.jackson.annotation.JsonProperty;").append(System.getProperty(SEPARATOR));
                    }
                    if (str.contains("@JsonProperty")) {
                        jsonProperty = str;
                    } else if (Objects.nonNull(jsonProperty)) {
                        System.out.println(line);
                        sb.append(jsonProperty).append(System.getProperty(SEPARATOR));
                        sb.append(str).append(System.getProperty(SEPARATOR));
                        jsonProperty = null;
                    } else if (str.contains("private") && str.contains("_")) {
                        System.out.println(line);
                        line++;
                        // 获取属性名
                        String memberName = getMemberName(str);
                        // 添加 JsonProperty 注解
                        sb.append("\t@JsonProperty(\"").append(memberName).append("\")").append(System.getProperty(SEPARATOR));
                        sb.append(modifyMemberName(str)).append(System.getProperty(SEPARATOR));
                    } else {
                        sb.append(str).append(System.getProperty(SEPARATOR));
                    }
                    str = bufferedReader.readLine();
                }
//            }
//            else {
//                String regex = "_[a-z]";
//                String str = bufferedReader.readLine();
//                while (Objects.nonNull(str)) {
//                    if (Pattern.matches(regex, str)) {
//                        str.replaceAll(regex, "");
//                    }
//                    str = bufferedReader.readLine();
//                }
//            }





            /*
            // 返回实体类（请自行确认返回实体类需不需要 JsonProperty 注解）
            else if (f.getName().contains("Vo")) {
                System.out.println(f.getName());
                // 修改的行
                int line = 0;
                String str = bufferedReader.readLine();
                while (Objects.nonNull(str)) {
                    line++;
                    if (str.contains("private") && str.contains("_")) {
                        System.out.println(line);
                        sb.append(modifyMemberName(str)).append(System.getProperty(SEPARATOR));
                    } else {
                        sb.append(str).append(System.getProperty(SEPARATOR));
                    }
                    str = bufferedReader.readLine();
                }
            } else if (f.getName().contains("EUmsServiceImpl")) {
                System.out.println(f.getName());
                // 修改的行
                int line = 0;
                // 是否括号结尾
                boolean isEndBraces = true;
                // 方法计数器，看是否在这个方法中
                boolean isInside = false;
                int currentMethod = 0;
                // 存储方法带 _ 的参数，一般不会超过两个
                Map<String, String> param = new HashMap<>(2);
                String str = bufferedReader.readLine();
                while (Objects.nonNull(str)) {
                    if (str.contains("{")) {
                        if (currentMethod == 0) {
                            isInside = true;
                        }
                        currentMethod++;
                    }
                    if (str.contains("}")) {
                        currentMethod--;
                    }
                    line++;
                    if (str.contains("private") || str.contains("public")) {
                        int paramBegin = str.indexOf("(") + 1;
                        if (!str.contains(")")) {
                            isEndBraces = false;
                        }
                        // 有下划线参数就不为空（方法名应该不会有下划线吧）
                        if (str.contains("_")) {
                            System.out.println(line);
                            String paramString = str.substring(paramBegin, isEndBraces ? str.indexOf(")") : str.length() - 1);
                            str = paramNameSetting(paramString, str, param);
                            sb.append(str).append(System.getProperty(SEPARATOR));
                        }

                    } else if (!isEndBraces) {
                        if (str.contains(")")) {
                            isEndBraces = true;
                        }
                        if (str.length() > 3) {
                            String paramString = str.trim().substring(0, isEndBraces ? str.indexOf(")") : str.length() - 1);
                            str = paramNameSetting(paramString, str, param);
                        }
                        sb.append(str).append(System.getProperty(SEPARATOR));
                    } else if(str.contains("_") && (str.contains(".get") || str.contains(".set"))) {


                        if (str.contains(".set")) {

                            str.substring(str.indexOf(".set"), str.indexOf("(", str.indexOf(".set")));
                        }


                        sb.append(str).append(System.getProperty(SEPARATOR));
                    } else {
                        sb.append(str).append(System.getProperty(SEPARATOR));
                    }

                    if (isInside && currentMethod == 0) {
                        isInside = false;
                        param.clear();
                    }
                    str = bufferedReader.readLine();
                }
            } else if (f.getName().contains("EUmsService")) {

            }
            */

            //将缓冲区数据写入文件中
            Writer writer = new FileWriter(new File(path + "\\" + f.getName()));
            writer.write(sb.toString());
            bufferedReader.close();
            fileReader.close();
            writer.close();
        }
    }

    private static String getMemberName(String str) {
        // 获取属性名
        int left, right;
        if (str.contains(">")) {
            left = str.indexOf("> ") + 2;
        } else {
            left = str.indexOf(' ', str.indexOf("private") + 10) + 1;
        }
        right = str.indexOf(';', left);
        return str.substring(left, right);
    }

    private static String getParamName(String str) {
        // 获取参数名
        String[] strings = str.split(" ");
        return strings[strings.length - 1];
    }

    private static String paramNameSetting(String paramString, String str, Map<String, String> param) {
        String[] params = paramString.split(",");
        for (String s : params) {
            String paramName = getParamName(s);
            String modifyParamName = modifyMemberName(paramName);
            // 替换参数名
            str = str.replace(paramName, modifyParamName);
            // 用来替换掉方法中用到该参数的
            param.put(paramName, modifyParamName);
        }
        return str;
    }

    /**
     * 修改属性名
     *
     * @param string
     * @return
     */
    private static String modifyMemberName(String string) {
        String str = string;
        // 单个属性中可能有多个下划线，将所有下划线和之后的字符替换
        while (StringUtils.isNotEmpty(str) && str.contains("_")) {
            String replaceBefore = str.substring(str.indexOf("_"), str.indexOf("_") + 2);
//            char c = (char)replaceBefore.toCharArray()[1] - 32;
            str = str.replace(replaceBefore,
                    String.valueOf(str.charAt(str.indexOf("_") + 1)).toUpperCase(Locale.ENGLISH));
        }
        return str;
    }

    /**
     * 删除作者信息
     *
     * @param path
     * @throws IOException
     */
    public static void deleteAuthor(String path) throws IOException {
        File file = new File(path);
        if (!file.exists()) {
            return;
        }
        File[] fs = file.listFiles();
        for (File f : fs) {
            if (f.isDirectory()) {
                deleteAuthor(f.getPath());
                continue;
            }
            FileReader fileReader = new FileReader(f);
            BufferedReader bufferedReader = new BufferedReader(fileReader);//把文件存到缓冲区
            StringBuilder sb = new StringBuilder();
            System.out.println(f.getName());
            // 只修改 java 文件
            if (!f.getName().endsWith("java")) {
                continue;
            }
            // 行数
            int line = 1;
            String str = bufferedReader.readLine();
            while (str != null) {
                line++;
                if (AuthorInfoEnum.getInstance(str)) {
                    System.out.println(line);
                } else {
                    sb.append(str).append(System.getProperty(SEPARATOR));
                }
                str = bufferedReader.readLine();
            }
            Writer writer = new FileWriter(new File(path + "\\" + f.getName()));//将缓冲区数据写入文件中
            writer.write(sb.toString());
            bufferedReader.close();
            fileReader.close();
            writer.close();
        }
    }

    /**
     * 为接口添加注解
     *
     * @throws IOException
     */
    public static void addAnnoation() throws IOException {
        File file = new File(PATH);
        File[] fs = file.listFiles();
        for (File f : fs) {
            FileReader fileReader = new FileReader(f);
            BufferedReader bufferedReader = new BufferedReader(fileReader);//把文件存到缓冲区
            Writer writer = new FileWriter(new File(PATH + "\\" + f.getName() + ".txt"));//将缓冲区数据写入文件中
            boolean flag = false;
            int line = 0;
            String str = bufferedReader.readLine();
            String copy = "";
            while (str != null) {
                System.out.println(++line);
                if (str.contains("@PostMapping") || str.contains("@PutMapping") || str.contains("@DeleteMapping")) {
                    copy = str;
                    flag = true;
                } else if (flag) {
                    int lastIndex = str.indexOf("(");
                    int firstIndex = str.lastIndexOf(" ", lastIndex) + 1;
                    String methodName = str.substring(firstIndex, lastIndex);
                    writer.write("\t@OperationType(\"" + methodName + "\")\r\n" + copy + "\r\n" + str + "\r\n");
                    flag = false;
                } else {
                    writer.write(str + "\r\n");
                }
                str = bufferedReader.readLine();
            }
            bufferedReader.close();
            fileReader.close();
            writer.close();
        }
    }

    /**
     * 生成账号加密信息
     */
    public static void createAccount() {
        System.out.println("13005412250".substring(0, 4) + SHA256Util.getSHA256String("13005412250"));
//        System.out.println("0123456255".substring(0, 4) + SHA256Util.getSHA256String("0123456255"));
//        System.out.println("0123456254".substring(0, 4) + SHA256Util.getSHA256String("0123456254"));

        JSONObject jsonObject = JSONObject.parseObject("{\"timestamp\":\"2020-09-27T01:44:52.114+0000\",\"status\":500,\"error\":\"InternalServerError\",\"message\":\"Failedtoparsemultipartservletrequest;nestedexceptionisjavax.servlet.ServletException:org.apache.tomcat.util.http.fileupload.FileUploadBase$InvalidContentTypeException:therequestdoesn'tcontainamultipart/form-dataormultipart/mixedstream,contenttypeheaderisnull\",\"path\":\"/v1/internal/eums/users/user-state\"}");
        System.out.println(jsonObject);
    }

    /**
     * 加载配置文件
     *
     * @return
     * @throws IOException
     */
    public static Properties loadProperties() throws IOException {
//        String configPath = "D:\\ProjectSpace\\IDEASpace\\test\\src\\work\\easyWork\\ePathConfig.properties";
        String configPath = "D:\\ProjectSpace\\IDEASpace\\Test\\src\\work\\config\\pathConfig.properties";
        Properties properties = new Properties();
        // 使用 BufferedReader 流读取 properties 文件
        BufferedReader bufferedReader = new BufferedReader(new FileReader(configPath));
        properties.load(bufferedReader);
        return properties;
    }
}

~~~
