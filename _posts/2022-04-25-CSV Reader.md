---
title: "CSV 읽어오기"
date: 2022-04-25 00:00:00 +0900
categories: [C#]
tags: [c#, unity]     # TAG names should always be lowercase
---

<span style="color:grey">*이 포스트는 이전 블로그에서 옮겨왔습니다.*</span>  

[GitHub 링크](https://github.com/Eunseo-1110/CSVtest.git)

 ```cs
using System.Collections.Generic;
using System.IO;
 
 
public static class CSVParse
{
    public static List<List<string>> Parse(string text)
    {
        StringReader stringReader = new StringReader(text);
        TextReader reader = stringReader;
        List<List<string>> result = new List<List<string>>();
        bool isQuote = false;
        bool isOneQuote = false;
        string line;
        List<string> tempLine = new List<string>();
        string tempStr = "";
 
        while ((line = reader.ReadLine()) != null)
        {
            foreach(char ch in line)
            {
                switch (ch)
                {
                    case ',':
                        if (isQuote)
                        {
                            tempStr += ch;
                        }
                        else
                        {
                            if (isOneQuote)
                            {
                                tempStr = tempStr.Remove(tempStr.Length - 1);
                                isOneQuote = false;
                            }
 
                            tempLine.Add(tempStr);
                            tempStr = "";
                        }
                        break;
                    case '"':
                        if (isQuote)
                        {
                            tempStr += ch;
                            isQuote = false;
                        }
                        else
                        {
                            isOneQuote = true;
                            isQuote = true;
                        }
                        break;
                    default:
                        tempStr += ch;
                        break;
                }
            }
 
            if (!isQuote)
            {
                if (isOneQuote)
                {
                    tempStr = tempStr.Remove(tempStr.Length - 1);
                    isOneQuote = false;
                }
                tempLine.Add(tempStr);
                result.Add(tempLine);
                tempLine = new List<string>();
                tempStr = "";
            }
            else
            {
                tempStr += "\n";
            }
        }
        return result;
    }
}
 ```

 csv읽는 함수. 라인별, 단어별로 읽어옴