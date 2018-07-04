#  打水

```C#
using System;
using System.Collections.Generic;
using System.Linq;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {


            //list.Add(new Pei("日本", 3.65));
            //list.Add(new Pei("波兰", 3.60));
            //list.Add(new Pei("和", 2.01));

            //double[] dlist = new double[] { 13,10,31,17,17,91,76,18,51,16,51,31 };

            //foreach (var item in dlist)
            //{
            //    list.Add(new Pei($"({item})", item));
            //}


            List<Pei> list = new List<Pei>();
            list.Add(new Pei("(波胆)日本 1-0  波兰", 7.5));
            list.Add(new Pei("(波胆)日本 0-0  波兰", 7.5));
            list.Add(new Pei("(波胆)日本 0-1  波兰", 7.5));
            list.Add(new Pei("(波胆)日本 1-1  波兰", 6));
            list.Add(new Pei("(波胆)日本 1-3  波兰", 11));
            list.Add(new Pei("(波胆)日本 0-1  波兰", 14));
            list.Add(new Pei("(波胆)日本 2-1  波兰", 11.5));
            list.Add(new Pei("(波胆)日本 2-0  波兰", 15.5));
            list.Add(new Pei("(波胆)日本 2-2  波兰", 16));
            List<Pei> list2 = new List<Pei>();
            list2.Add(new Pei("(波胆)日本 3-0  波兰", 46));
            list2.Add(new Pei("(波胆)日本 0-3  波兰", 41));
            list2.Add(new Pei("(波胆)日本 3-1  波兰", 36));
            list2.Add(new Pei("(波胆)日本 1-3  波兰", 26));
            list2.Add(new Pei("(波胆)日本 3-2  波兰", 46));
            list2.Add(new Pei("(波胆)日本 2-3  波兰", 36));
            List<Pei> list3 = new List<Pei>();
            list3.Add(new Pei("(让球)日本 - 波兰 0", 2, 0.4));
            list3.Add(new Pei("(让球)日本 - 波兰  ", 1.91, 0.6));

            List<Pei> list4 = new List<Pei>();
            list4.Add(new Pei("(独赢)日本 - 波兰  胜", 2.8, 0.4));
            list4.Add(new Pei("(独赢)日本 - 波兰  平", 3.15, 0.2));
            list4.Add(new Pei("(独赢)日本 - 波兰  负", 2.72, 0.6));
            //GetPl(list);
            //GetPl(list2);
            //GetPl(list3);
            //GetPl(list4);
            List<Pei> list5 = new List<Pei>();
            list5.Add(new Pei("(波胆)日本 1-0  波兰", 8.3));
            list5.Add(new Pei("(波胆)日本 0-0  波兰", 8));
            list5.Add(new Pei("(波胆)日本 0-1  波兰", 8.0));
            list5.Add(new Pei("(波胆)日本 1-1  波兰", 6.5));

            List<Pei> list6 = new List<Pei>();
            list6.Add(new Pei("(波胆)日本 2-0  波兰", 16));
            list6.Add(new Pei("(波胆)日本 0-2  波兰", 15));
            list6.Add(new Pei("(波胆)日本 2-1  波兰", 11));
            list6.Add(new Pei("(波胆)日本 1-2  波兰", 10));
            list6.Add(new Pei("(波胆)日本 2-2  波兰", 15));
            var l = GetPl("冷门", list5, 100);
            var r = GetPl("热门", list6, 150);


            List<Pei> list7= new List<Pei>();
            list7.Add(l);
            list7.Add(r);
            GetPl("冷热打水", list7, 620);
            list5.AddRange(list6);
            GetPl("冷热合买", list5, 100);
            Console.ReadKey();
        }

        private static Pei GetPl(string name, List<Pei> list, double minAmount)
        {
            Console.WriteLine($"============{name}===============");
            double jiben = list.Count * list.Count;
            double zongpei = list.Sum(x => x.P);
            Console.WriteLine($"总下注方案：{list.Count}");
            Console.WriteLine($"基本赔率点：{jiben}");
            Console.WriteLine($"总赔率点：{zongpei}");
            Console.WriteLine($"方案：{(zongpei > jiben ? "符合" : "不符合") }");

            Console.WriteLine($"最低下注金额：{minAmount}");
            double maxpei = list.Max(x => x.P); //最大赔率
            double maxpeiYin = maxpei * minAmount;  //最大赔率 盈利
            Console.WriteLine($"最大返点：{list.Max(x => x.P)}，下注{minAmount}  可赢： {maxpeiYin}");
            double amou = list.Sum(x => x.Bu(maxpeiYin));
            Console.WriteLine($"总下注金额:{amou}    盈亏：{maxpeiYin - amou }  ");

            Console.WriteLine($"=>总基本赔率（{maxpeiYin / amou}）   可享受反水：{amou * 0.025}");

            return new Pei(name, maxpeiYin / amou);
        }
    }
    class Pei
    {
        public Pei(string name, double p)
        {
            Name = name;
            P = p;
        }
        public Pei(string name, double p, double q)
        {
            Name = name;
            P = p;
            Qiwan = q;
        }
        public string Name { get; set; }
        public double P { get; set; }

        public double Qiwan { get; set; }

        public double Xia(double amount)
        {
            return P * amount;
        }

        public double Bu(double amount)
        {
            var xx = Math.Ceiling(amount / P);
            Console.WriteLine($"{Name}  下注金额：{xx}  概率({(1 / P).ToString("0.00%")}) （{P}） 盈利：{Xia(xx)}");
            //Console.WriteLine($" 期望数值({Qiwan * P})   ");
            return xx;
        }
    }

}


```