直接上代码：
```csharp
using System;
using System.Globalization;
using System.Text;
using System.Web;

namespace TestEncoder
{
    /// <summary>
    /// 字符串安全编码
    /// </summary>
    public class URLEncoder
    {
        private const string ENCODING_UTF8 = "UTF-8";

        public static string Encode(String value)
        {
            return HttpUtility.UrlEncode(value, Encoding.UTF8);
        }

        public static string PercentEncode(String value)
        {
            // 不同语言不一样
            StringBuilder stringBuilder = new StringBuilder();
            string text = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-_.~";
            byte[] bytes = Encoding.GetEncoding(ENCODING_UTF8).GetBytes(value);

            foreach (char c in bytes)
            {
                if (text.IndexOf(c) >= 0)
                {
                    stringBuilder.Append(c);
                }
                else
                {
                    stringBuilder.Append("%").Append(
                        string.Format(CultureInfo.InvariantCulture, "{0:X2}", (int)c));
                }
            }
            return stringBuilder.ToString();
        }
    }
}

```
> 关键在于编码后的 `%` 后的 2 位字母为大写