Для "лицензирования" Confluence 6.3.4 необходимо внести изменения в два файла (приведены актуальные версии файлов на момент установки):
   
```
     atlassian-extras-decoder-v2-3.2.jar
     atlassian-universal-plugin-manager-plugin-2.22.5.jar
``` 

Месторасположение файлов соответственно:
   
```
     /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-3.2.jar
     /opt/atlassian/confluence/confluence/WEB-INF/atlassian-bundled-plugins/atlassian-universal-plugin-manager-plugin-2.22.5.jar
```

Можно использовать уже пропатченный для Jira atlassian-universal-plugin-manager-plugin-2.22.5.jar или приготовить его "с нуля" - см. Readme.md в каталоге jira

Здесь рассмотрим только патч файла atlassian-extras-decoder-v2-3.2.jar.

- скачиваем на локальный компьютер файлы atlassian-extras-common-3.2.jar, atlassian-extras-decoder-api-3.2.jar, atlassian-extras-api-3.2.jar и atlassian-extras-decoder-v2-3.2.jar из каталога /opt/atlassian/confluence/confluence/WEB-INF/lib/
- устанавливаем инструмент JD-GUI (http://jd.benow.ca/). Далее:

   - открываем atlassian-extras-decoder-v2-3.2.jar с помощью декомпилятора JD-GUI
   - жмем File -> Save All Sources или Ctrl+Alt+S (сохранится архив atlassian-extras-decoder-v2-3.2.jar.src.zip)
   - распаковываем полученный архив с помощью архиватора
   - в atlassian-extras-decoder-v2-3.2.jar.src/com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java находим метод loadLicenseConfiguration (в моем случае выглядел так):

```
     /*     */   private Properties loadLicenseConfiguration(Reader text)
     /*     */   {
     /*     */     try
     /*     */     {
     /* 218 */       Properties props = new Properties();
     /* 219 */       new DefaultPropertiesPersister().load(props, text);
     /* 220 */       return props;
     /*     */     }
     /*     */     catch (IOException e)
     /*     */     {
     /* 224 */       throw new LicenseException("Could NOT load properties from reader", e);
     /*     */     }
     /*     */   }

```

   - и добавляем в данный метод информацию о лицензии:

```
     /*     */   private Properties loadLicenseConfiguration(Reader text)
     /*     */   {
     /*     */     try
     /*     */     {
     /* 218 */       Properties props = new Properties();
     /* 219 */       new DefaultPropertiesPersister().load(props, text);

                     props.setProperty("LicenseExpiryDate", "2031-01-01");
                     props.setProperty("MaintenanceExpiryDate", "2031-01-01");
                     props.setProperty("Evaluation", "false");
                     props.setProperty("NumberOfUsers", "-1");
                     props.setProperty("Organisation", "TerriconMembers");
                     props.setProperty("PurchaseDate", "2017-01-01");
                     props.setProperty("SEN", "SEN-L10303078");

     /* 220 */       return props;
     /*     */     }
     /*     */     catch (IOException e)
     /*     */     {
     /* 224 */       throw new LicenseException("Could NOT load properties from reader", e);
     /*     */     }
     /*     */   }

```

   - сохраняем файл
   - копируем файлы atlassian-extras-common-3.2.jar, atlassian-extras-decoder-api-3.2.jar, atlassian-extras-api-3.2.jar и commons-codec-1.9.jar в директорию с исходниками (atlassian-extras-decoder-v2-3.2.jar.src)
   - переходим в каталог с исходниками (atlassian-extras-3.2.jar.src) и компилируем класс из java-файла, который мы правили:

```
     javac -cp commons-codec-1.9.jar:atlassian-extras-common-3.2.jar:atlassian-extras-decoder-api-3.2.jar:atlassian-extras-api-3.2.jar -sourcepath ./ com/atlassian/extras/decoder/v2/Version2LicenseDecoder.java
```
   - могут быть ошибки (вызваны “кривостью” декомпиляции исходников), устраняем их и еще раз компилируем класс
   - после успешного выполнения в каталоге com/atlassian/extras/decoder/v2/ появится файл Version2LicenseDecoder.class
   - полученныей файл копируем с заменой (по такому же пути com/atlassian/extras/decoder/v2/) в архив atlassian-extras-decoder-v2-3.2.jar (проще всего это сделать через mc - Midnight Commander)
   - в общем случае “новый” архив atlassian-extras-decoder-v2-3.2.jar необходимо положить (с заменой) на сервере с Confluence в каталог /opt/atlassian/confluence/confluence/WEB-INF/lib/, удалить содержимое каталогов ${CONFLUENCE_HOME}/plugins-osgi-cache/transformed-plugins/* и ${CONFLUENCE_HOME}/plugins-osgi-cache/felix/* после чего перезапустить confluence, в нашем случае нужно пересобрать docker-образ согласно инструкциям в Dockerfile