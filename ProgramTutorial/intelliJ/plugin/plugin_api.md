关于当前编辑的文件  
```
extends AnAction{

    @Override
    public void actionPerformed(AnActionEvent e) {
   
   
        //  getProjectFilePath=/home/alex/WorkSpace/AndroidStudio/MyApplication/.idea/misc.xml  
        //  project.getProjectFilePath()  
        
        //  /home/alex/WorkSpace/AndroidStudio/MyApplication
        project.getBasePath();
        
        //  MyApplication
        project.getName();  
        
        //  /home/alex/WorkSpace/AndroidStudio/MyApplication/.idea/workspace.xml
        project.getWorkspaceFile();  
         
        //  鼠标在文件上的右键属性  
        PsiFile psiFile = e.getData(LangDataKeys.PSI_FILE);
        //  MainActivity.kt
        //  psiFile.getName()

        //  MainActivity.kt
        //  psiFile.getVirtualFile().getName()

        //  /home/alex/WorkSpace/AndroidStudio/MyApplication/app/src/main/java/com/example/myapplication/MainActivity.kt
        //  psiFile.getVirtualFile().getPath()

        //  /home/alex/WorkSpace/AndroidStudio/MyApplication/app/src/main/java/com/example/myapplication/module/login
        //  psiFile.getVirtualFile().getParent().getPath()

        //  LoginActivity
        //  psiFile.getVirtualFile().getNameWithoutExtension()
   
   
       //  鼠标在文件/文件夹 上的右键属性  
       //  鼠标右键所选择的路径
       IdeView ideView = actionEvent.getRequiredData(LangDataKeys.IDE_VIEW);
       PsiDirectory directory = ideView.getOrChooseDirectory();
       //  entity  
       directory.getName();
       
       //  /home/alex/WorkSpace/AndroidStudio/MyApplication/app/src/main/java/com/alex/kooper/list/entity
       directory.getVirtualFile().getPath();         
       
    }

}

```

添加一个方法  
```
String methodText = buildMethodText(className);
PsiMethod psiMethod = elementFactory.createMethodFromText(methodText, psiClass);
psiClass.add(psiMethod);


private String buildMethodText (String className){
    return "public static " + className + " getInstance() {\n" +
            "        return " + buildFiledText() + ";\n" +
            "    }";
}
```

创建内部类  
```
AnActionEvent e;  
Editor editor = e.getData(PlatformDataKeys.EDITOR);
PsiElementFactory elementFactory = JavaPsiFacade.getElementFactory(project);

PsiClass innerClass = elementFactory.createClass(innerClassName);
PsiModifierList classModifierList = innerClass.getModifierList();
if (classModifierList != null) {
    classModifierList.setModifierProperty(PsiModifier.PRIVATE, true);
    classModifierList.setModifierProperty(PsiModifier.STATIC, true);
}
psiClass.add(innerClass);
```
https://github.com/wangyiwy/Singleton-Plugin/blob/401c4622fdc37cb18114c8dae1d40b9693afc758/src/wangyi/plugin/singleton/builder/InnerClassPatternBuilder.java  

获取当前编辑的class对象  
```
PsiElement element = psiFile.findElementAt(editor.getCaretModel().getOffset());
PsiClass psiClass = PsiTreeUtil.getParentOfType(element, PsiClass.class);
```
格式化代码  
CodeStyleManager.getInstance(project).reformat(psiClass);  

//创建接口
elementFactory.createInterface("TestInterface");  
