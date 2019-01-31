## 邮件发送demo


1.pom配置
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

2.邮件发送
```

@Autowired
private JavaMailSender mailSender;


//freemaker模板
@Autowired
private FreeMarkerConfigurer freeMarkerConfigurer;

/**
 * 邮件发送
 *
 * @param tos            收件人
 * @param ccs            抄送人
 * @param filePaths      附件
 * @param inlineImgPaths 图片
 * @param subject        主题
 * @param model          参数
 * @param templateName   模板
 * @throws Exception
 */
public void sendPointsEmail(String[] tos, String[] ccs, List<String> filePaths, List<Map<String, String>> inlineImgPaths, String subject,
                            Map<String, Object> model, String templateName) throws Exception {
    MimeMessage mimeMessage = mailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);

    helper.setFrom("邮件发送demo");
    helper.setTo(tos);
    //抄送人
    if (!(ccs == null || ccs.length == 0)) {
        helper.setCc(ccs);
    }
    //附件
    if (filePaths != null &&
            !filePaths.isEmpty()) {
        for (String filePath : filePaths) {
            DataSource ds = new FileDataSource(filePath);
            // 添加到附件中
            helper.addAttachment(MimeUtility.encodeWord(ds.getName()), ds);

        }
    }
    //图片
    if (inlineImgPaths != null && !inlineImgPaths.isEmpty()) {
        for (Map<String, String> imgPathMap : inlineImgPaths) {
            FileSystemResource fileImg = new FileSystemResource(new File(imgPathMap.get("filePath")));
            helper.addInline(imgPathMap.get("fileKey"), fileImg);
        }
    }
    helper.setSubject(subject);

    Template template = freeMarkerConfigurer.getConfiguration().getTemplate(templateName);
    String text = FreeMarkerTemplateUtils.processTemplateIntoString(template, model);
    helper.setText(text, true);

    mailSender.send(mimeMessage);
  
}

```