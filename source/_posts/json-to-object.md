---
title: json转换成C#类
date: 2022-05-11 10:42:12
tags: 技术杂谈
---

## 应用场景

最近碰到一个问题，就是想把json字符串中的字段名称都改成首字母小写，当然这个json是非常大的，手动改不理智，那有没有什么办法通过什么方式直接将json字符串的首字母都改成小写的呢？一开始是想通过Newtonsoft.Json.dll这个类库，直接反序列化为动态的类(dynamic)，然后重新序列化成json字符串时指定：`ContractResolver = new CamelCasePropertyNamesContractResolver()`，但是还是行不通。如果我们有实现定义好的实体类，再序列化的时候通过上面的设置是可以输出首字母小写的json字符串，基于这个考虑，是否可以通过原json字符串然后动态生成实体类，再通过反序列化成该实体类，最后把得到的对象再序列化成json字符串，是否可达目的呢。请往下看。

## 动态生成实体类

- 创建一个控制台项目：`dotnet new console -o JsonToObject`

- 新建一个json文件，里面就是需要转换的json内容

- 读取json内容，拼接json中涉及到的所有的类的定义

  ```C#
  Action<string> Write = Console.WriteLine;
  var jsonString = File.ReadAllText("demo.json");//读取json文件里的内容
  var jObject = JObject.Parse(jsonString);//Newtonsoft.Json中的JObject.Parse转换成json对象
  
  Dictionary<string, string> classDicts = new Dictionary<string, string>();//key为类名，value为类中的所有属性定义的字符串
  classDicts.Add("Root", GetClassDefinion(jObject));//拼接顶层的类
  foreach (var item in jObject.Properties())
  {
      classDicts.Add(item.Name, GetClassDefinion(item.Value));
      GetClasses(item.Value, classDicts);
  }
  //下面是将所有的类定义完整拼接起来
  StringBuilder sb = new StringBuilder(1024);
  sb.AppendLine("using System;");
  sb.AppendLine("using System.Collections.Generic;");
  sb.AppendLine("namespace JsonToObject");
  sb.AppendLine("{");
  foreach (var item in classDicts)
  {
      sb.Append($"public class {item.Key}" + Environment.NewLine);
      sb.Append("{" + Environment.NewLine);
      sb.Append(item.Value);
      sb.Append("}" + Environment.NewLine);
  }
  sb.AppendLine("}");
  Write(sb.ToString());
  
  //递归遍历json节点，把需要定义的类存入classes
  void GetClasses(JToken jToken, Dictionary<string, string> classes)
  {
      if (jToken is JValue)
      {
          return;
      }
      var childToken = jToken.First;
      while (childToken != null)
      {
          if (childToken.Type == JTokenType.Property)
          {
              var p = (JProperty)childToken;
              var valueType = p.Value.Type;
  
              if (valueType == JTokenType.Object)
              {
                  classes.Add(p.Name, GetClassDefinion(p.Value));
                  GetClasses(p.Value, classes);
              }
              else if (valueType == JTokenType.Array)
              {
                  foreach (var item in (JArray)p.Value)
                  {
                      if (item.Type == JTokenType.Object)
                      {
                          if (!classes.ContainsKey(p.Name))
                          {
                              classes.Add(p.Name, GetClassDefinion(item));
                          }
  
                          GetClasses(item, classes);
                      }
                  }
              }
          }
  
          childToken = childToken.Next;
      }
  }
  
  //获取类中的所有的属性
  string GetClassDefinion(JToken jToken)
  {
      StringBuilder sb = new(256);
      var subValueToken = jToken.First();
      while (subValueToken != null)
      {
          if (subValueToken.Type == JTokenType.Property)
          {
              var p = (JProperty)subValueToken;
              var valueType = p.Value.Type;
              if (valueType == JTokenType.Object)
              {
                  sb.Append("public " + p.Name + " " + p.Name + " {get;set;}" + Environment.NewLine);
              }
              else if (valueType == JTokenType.Array)
              {
                  var arr = (JArray)p.Value;
                  //a.First
  
                  switch (arr.First().Type)
                  {
                      case JTokenType.Object:
                          sb.Append($"public List<{p.Name}> " + p.Name + " {get;set;}" + Environment.NewLine);
                          break;
                      case JTokenType.Integer:
                          sb.Append($"public List<int> " + p.Name + " {get;set;}" + Environment.NewLine);
                          break;
                      case JTokenType.Float:
                          sb.Append($"public List<float> " + p.Name + " {get;set;}" + Environment.NewLine);
                          break;
                      case JTokenType.String:
                          sb.Append($"public List<string> " + p.Name + " {get;set;}" + Environment.NewLine);
                          break;
                      case JTokenType.Boolean:
                          sb.Append($"public List<bool> " + p.Name + " {get;set;}" + Environment.NewLine);
                          break;
                      default:
                          break;
                  }
              }
              else
              {
                  switch (valueType)
                  {
                      case JTokenType.Integer:
                          sb.Append($"public int " + p.Name + " {get;set;}" + Environment.NewLine);
                          break;
                      case JTokenType.Float:
                          sb.Append($"public float " + p.Name + " {get;set;}" + Environment.NewLine);
                          break;
                      case JTokenType.String:
                          sb.Append($"public string " + p.Name + " {get;set;}" + Environment.NewLine);
                          break;
                      case JTokenType.Boolean:
                          sb.Append($"public bool " + p.Name + " {get;set;}" + Environment.NewLine);
                          break;
                      default:
                          break;
                  }
              }
          }
  
          subValueToken = subValueToken.Next;
      }
  
      return sb.ToString();
  }
  ```

  - 现在有了类的定义字符串，接下来就可以动态编译成实体类

    ```C#
    Write("Let's compile!");
    Write("Parsing the code into the SyntaxTree");
    SyntaxTree syntaxTree = CSharpSyntaxTree.ParseText(sb.ToString());
    
    string assemblyName = Path.GetRandomFileName();
    var refPaths = new[] {
                    typeof(object).GetTypeInfo().Assembly.Location,
                    typeof(Console).GetTypeInfo().Assembly.Location,
                    Path.Combine(Path.GetDirectoryName(typeof(System.Runtime.GCSettings).GetTypeInfo().Assembly.Location), "System.Runtime.dll")
                };
    MetadataReference[] references = refPaths.Select(r => MetadataReference.CreateFromFile(r)).ToArray();
    
    Write("Adding the following references");
    foreach (var r in refPaths)
        Write(r);
    
    Write("Compiling ...");
    CSharpCompilation compilation = CSharpCompilation.Create(
        assemblyName,
        syntaxTrees: new[] { syntaxTree },
        references: references,
        options: new CSharpCompilationOptions(OutputKind.DynamicallyLinkedLibrary));
    
    using (var ms = new MemoryStream())
    {
        EmitResult result = compilation.Emit(ms);
    
        if (!result.Success)
        {
            Write("Compilation failed!");
            IEnumerable<Diagnostic> failures = result.Diagnostics.Where(diagnostic =>
                diagnostic.IsWarningAsError ||
                diagnostic.Severity == DiagnosticSeverity.Error);
    
            foreach (Diagnostic diagnostic in failures)
            {
                Console.Error.WriteLine("\t{0}: {1}", diagnostic.Id, diagnostic.GetMessage());
            }
        }
        else
        {
            Write("Compilation successful! Now instantiating and executing the code ...");
            ms.Seek(0, SeekOrigin.Begin);
    
            Assembly assembly = AssemblyLoadContext.Default.LoadFromStream(ms);
            var type = assembly.GetType("JsonToObject.Root");
            var instance = assembly.CreateInstance("JsonToObject.Root");
    	    //反射获取静态的 DeserializeObject方法
            var deserializeObject = typeof(JsonConvert).GetGenericMethod("DeserializeObject", BindingFlags.Public | BindingFlags.Static, new Type[] { typeof(string), typeof(JsonSerializerSettings) });
            var genericDeserializeObject = deserializeObject.MakeGenericMethod(type);
    		//执行反序列化
            var root = genericDeserializeObject.Invoke(null, new object[] { jsonString, null });
            //输出序列化的结果
            Write(JsonConvert.SerializeObject(root, new JsonSerializerSettings { ContractResolver = new CamelCasePropertyNamesContractResolver() }));
        }
    }
    ```

    通过反射获取静态方法的扩展

    ```C#
    public static class Extension
    {
        public static MethodInfo GetGenericMethod(this Type targetType, string name, BindingFlags flags, params Type[] parameterTypes)
        {
            var methods = targetType.GetMethods(flags).Where(m => m.Name == name && m.IsGenericMethod);
            var flag = false;
            foreach (MethodInfo method in methods)
            {
                var parameters = method.GetParameters();
                if (parameters.Length != parameterTypes.Length)
                    continue;
    
                for (var i = 0; i < parameters.Length; i++)
                {
                    if (parameters[i].ParameterType != parameterTypes[i])
                    {
                        break;
                    }
                    if (i == parameters.Length - 1)
                    {
                        flag = true;
                    }
                }
                if (flag)
                {
                    return method;
                }
            }
            return null;
        }
    }
    ```

    至此，我们已经显示了，文章开头说的将json字符串的首字母都变成小写的。

    ## 总结

    本文主要涉及以下的内容

    - 如何通过json字符串拼接类的定义

    - 如何通过动态编译的方式动态生成类

    - 反射动态调用反序列化的方法

      希望本文能够对大家有所帮助，完整地项目地址可以参考：[dynamic_demo](https://github.com/likeshan168/test_demo/tree/main/dynamic_demo)项目

