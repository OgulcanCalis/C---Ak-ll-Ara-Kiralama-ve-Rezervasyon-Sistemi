using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Text;

namespace AracKiralamaProjesi     //C# Console ile Akıllı Araç Kiralama ve Rezervasyon Sistemi
{
    // Araç Bilgileri
    public class Arac
    {
        public string Plaka { get; set; }
        public string MarkaModel { get; set; }
        public double GunlukFiyat { get; set; }
        public string Sinif { get; set; } // SUV, Sedan, Ekonomi
    }

    // Rezervasyon Bilgileri
    public class Rezervasyon
    {
        public string Musteri { get; set; }
        public string Plaka { get; set; }
        public DateTime Baslangic { get; set; }
        public DateTime Bitis { get; set; }
        public double ToplamUcret { get; set; }
    }

    class Program
    {
        static List<Arac> araclar = new List<Arac>();   //araclar adında liste fonksiyonunu oluşturduk.
        static List<Rezervasyon> rezervasyonlar = new List<Rezervasyon>();   //rezervasyonlar adında rezervasyon için liste fonksiyonunu oluşturduk.

        static void Main(string[] args)
        {
            Console.OutputEncoding = Encoding.UTF8;
            Console.InputEncoding = Encoding.UTF8;
            CultureInfo.DefaultThreadCurrentCulture = new CultureInfo("tr-TR");
            CultureInfo.DefaultThreadCurrentUICulture = new CultureInfo("tr-TR");

            // Başlangıç verileri yüklenir
            VarsayilanVerileriYukle();

            while (true)
            {
                MenuGoster();
                string secim = Console.ReadLine();
                IslemYap(secim);
            }
        }

        static void MenuGoster()
        {
            Console.Clear();
            Console.WriteLine("=========================================================================================");
            Console.WriteLine("                  AKILLI ARAÇ KİRALAMA ve REZERVASYON SİSTEMİ                            ");
            Console.WriteLine("=========================================================================================");

            Console.ForegroundColor = ConsoleColor.Green; Console.WriteLine("1) Araçları Listele (Saatlik/Günlük/Aylık Tablo)");
            Console.ForegroundColor = ConsoleColor.DarkYellow; Console.WriteLine("2) Araç Sınıfına Göre Listele (SUV/Sedan vb.)");
            Console.ForegroundColor = ConsoleColor.Magenta; Console.WriteLine("3) Müsait Durumda Olan Araçları Göster (Tarih Sorgulu)");
            Console.ForegroundColor = ConsoleColor.Cyan; Console.WriteLine("4) Rezervasyon Ekle");
            Console.ForegroundColor = ConsoleColor.Red; Console.WriteLine("5) Rezervasyon İptal Et");
            Console.ForegroundColor = ConsoleColor.Blue; Console.WriteLine("6) Müşteri Rezervasyon Geçmişi");
            Console.ForegroundColor = ConsoleColor.Yellow; Console.WriteLine("7) Rezervasyon Ara (Müşteri/Plaka)");
            Console.ForegroundColor = ConsoleColor.Gray; Console.WriteLine("8) Toplam Gelir ve İstatistik");
            Console.ForegroundColor = ConsoleColor.DarkMagenta; Console.WriteLine("9) En Çok Kiralanan Araç");
            Console.ForegroundColor = ConsoleColor.White; Console.WriteLine("10) Verileri Kaydet (JSON/TXT)");
            Console.ForegroundColor = ConsoleColor.DarkRed; Console.WriteLine("11) Çıkış");

            Console.ResetColor();
            Console.WriteLine("=========================================================================================");
            Console.Write("Seçiminiz: ");
        }

        static void IslemYap(string secim)
        {
            Console.WriteLine();
            try
            {
                switch (secim)
                {
                    case "1": AraclariListele(); break;
                    case "2": SinifaGoreListele(); break;
                    case "3": MusaitAraclariSorgula(); break; // İstenen 3. Seçenek
                    case "4": RezervasyonEkle(); break;
                    case "5": RezervasyonIptal(); break;
                    case "6": MusteriGecmisi(); break;
                    case "7": RezervasyonAra(); break;
                    case "8": GelirRaporu(); break;
                    case "9": EnCokKiralananArac(); break;
                    case "10": VerileriKaydet(); break;
                    case "11": Environment.Exit(0); break;
                    default: Console.WriteLine("Geçersiz seçim yaptınız!"); break;
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("Hata oluştu: " + ex.Message);
            }
            Console.WriteLine("\nDevam etmek için bir tuşa basın...");
            Console.ReadKey();
        }

        // --- 1. FONKSİYON: DETAYLI FİYAT TABLOSU ---
        static void AraclariListele()
        {
            Console.WriteLine("\n{0,-12} {1,-18} {2,-10} {3,-12} {4,-12} {5,-12}", "Plaka", "Marka/Model", "Sınıf", "Saatlik", "Günlük", "Aylık (x30)");
            Console.WriteLine(new string('-', 85));
            foreach (var a in araclar)
            {
                double saatlik = a.GunlukFiyat / 24;
                double aylik = a.GunlukFiyat * 30 * 0.8; // Aracın %20 indirimli aylık fiyatı
                Console.WriteLine("{0,-12} {1,-18} {2,-10} {3,-12:C2} {4,-12:C2} {5,-12:C2}",
                    a.Plaka, a.MarkaModel, a.Sinif, saatlik, a.GunlukFiyat, aylik);
            }
        }

        // --- 2. FONKSİYON: SINIF FİLTRELEME ---
        static void SinifaGoreListele()
        {
            Console.Write("Listelenecek Sınıfı Girin (SUV/Sedan/Ekonomi): ");
            string sinif = Console.ReadLine();
            var filtreli = araclar.Where(a => a.Sinif.Equals(sinif, StringComparison.OrdinalIgnoreCase));
            foreach (var a in filtreli) Console.WriteLine($"> {a.Plaka} - {a.MarkaModel}");
        }

        // --- 3. FONKSİYON: MÜSAİT ARAÇ SORGULAMA ---
        static void MusaitAraclariSorgula()
        {
            Console.Write("Başlangıç Tarihi (GG.AA.YYYY): ");
            DateTime bas = DateTime.Parse(Console.ReadLine());
            Console.Write("Bitiş Tarihi (GG.AA.YYYY): ");
            DateTime bit = DateTime.Parse(Console.ReadLine());

            // Rezervasyonu olan plakaları bul
            var doluPlakalar = rezervasyonlar
                .Where(r => (bas < r.Bitis && bit > r.Baslangic))
                .Select(r => r.Plaka).ToList();

            var AracMusaitMi = araclar.Where(a => !doluPlakalar.Contains(a.Plaka)).ToList();

            Console.WriteLine($"\n{bas.ToShortDateString()} - {bit.ToShortDateString()} Arası Müsait Araçlar:");
            if (AracMusaitMi.Count == 0) Console.WriteLine("Üzgünüz, bu tarihlerde tüm araçlar dolu.");
            foreach (var a in AracMusaitMi) Console.WriteLine($"- {a.Plaka} | {a.MarkaModel} ({a.Sinif})");
        }

        // --- 4. FONKSİYON: REZERVASYON EKLE (GELİR GARANTİLİ) ---
        static void RezervasyonEkle()
        {
            Console.Write("Müşteri Ad Soyad: "); string musteri = Console.ReadLine();
            Console.Write("Kiralanacak Araç Plakası: "); string plaka = Console.ReadLine();
            Console.Write("Kiralama Başlangıç: "); DateTime bas = DateTime.Parse(Console.ReadLine());
            Console.Write("Kiralama Bitiş: "); DateTime bit = DateTime.Parse(Console.ReadLine());

            var arac = araclar.FirstOrDefault(a => a.Plaka.Equals(plaka, StringComparison.OrdinalIgnoreCase));
            if (arac == null) { Console.WriteLine("Araç bulunamadı!"); return; }

            // Çakışma kontrolü
            bool doluMu = rezervasyonlar.Any(r => r.Plaka.Equals(plaka, StringComparison.OrdinalIgnoreCase) && (bas < r.Bitis && bit > r.Baslangic));

            if (!doluMu)
            {
                double gun = (bit - bas).TotalDays;
                if (gun <= 0) gun = 1; // 0 TL hatasını önlemek için en az 1 gün sayılır

                double toplam = gun * arac.GunlukFiyat;
                rezervasyonlar.Add(new Rezervasyon { Musteri = musteri, Plaka = plaka, Baslangic = bas, Bitis = bit, ToplamUcret = toplam });

                Console.ForegroundColor = ConsoleColor.Cyan;
                Console.WriteLine($"İşlem Başarılı! Toplam Gün: {gun:N0}, Tahsil Edilen: {toplam:C2}");
                Console.ResetColor();
            }
            else Console.WriteLine("HATA: Araç bu tarihlerde zaten kiralanmıştır.");
        }

        // --- 5. FONKSİYON: İPTAL ET ---
        static void RezervasyonIptal()
        {
            Console.Write("İptal Edilecek Rezervasyonun Plakasını Giriniz: ");
            string plaka = Console.ReadLine();
            var silinecek = rezervasyonlar.FirstOrDefault(r => r.Plaka.Equals(plaka, StringComparison.OrdinalIgnoreCase));
            if (silinecek != null)
            {
                rezervasyonlar.Remove(silinecek);
                Console.WriteLine("Rezervasyon sistemden silindi.");
            }
            else Console.WriteLine("Kayıt bulunamadı.");
        }

        // --- 8. FONKSİYON: TOPLAM GELİR ---
        static void GelirRaporu()
        {
            double toplam = rezervasyonlar.Sum(r => r.ToplamUcret);
            Console.WriteLine("-------------------------------------------");
            Console.WriteLine($"Aktif Rezervasyon Sayısı : {rezervasyonlar.Count}");
            Console.WriteLine($"Toplam Kasa Bakiyesi      : {toplam:C2}");
            Console.WriteLine("-------------------------------------------");
        }

        // --- 9. FONKSİYON: EN ÇOK KİRALANAN ---
        static void EnCokKiralananArac()
        {
            var rapor = rezervasyonlar.GroupBy(r => r.Plaka)
                        .OrderByDescending(g => g.Count())
                        .FirstOrDefault();
            if (rapor != null) Console.WriteLine($"Zirvedeki Araç: {rapor.Key} (Toplam {rapor.Count()} kiralama)");
            else Console.WriteLine("Henüz veri yok.");
        }

        static void VerileriKaydet()
        {
            using (StreamWriter sw = new StreamWriter("SistemRaporu.txt"))
            {
                sw.WriteLine("--- GÜNCEL REZERVASYON LİSTESİ ---");
                foreach (var r in rezervasyonlar)
                    sw.WriteLine($"{r.Musteri} | {r.Plaka} | {r.ToplamUcret:C2}");
            }
            Console.WriteLine("Veriler SistemRaporu.txt dosyasına yazıldı.");
        }

        static void VarsayilanVerileriYukle()
        {
            araclar.Add(new Arac { Plaka = "34RENT01", MarkaModel = "Mercedes C200", GunlukFiyat = 4500, Sinif = "Sedan" });
            araclar.Add(new Arac { Plaka = "06RENT02", MarkaModel = "Volvo XC90", GunlukFiyat = 7500, Sinif = "SUV" });
            araclar.Add(new Arac { Plaka = "35RENT03", MarkaModel = "Fiat Egea", GunlukFiyat = 1200, Sinif = "Ekonomi" });
            araclar.Add(new Arac { Plaka = "34ABC123", MarkaModel = "Toyota Corolla", GunlukFiyat = 2300, Sinif = "Sedan" });
            araclar.Add(new Arac { Plaka = "35VHL6846", MarkaModel = "Renault Duster", GunlukFiyat = 18000, Sinif = "SUV" });
            araclar.Add(new Arac { Plaka = "07CNS581", MarkaModel = "SKODA Superb", GunlukFiyat = 8800, Sinif = "Sedan" });
            araclar.Add(new Arac { Plaka = "06AWZ219", MarkaModel = "SKODA Octavia", GunlukFiyat = 7000, Sinif = "Sedan" });
            araclar.Add(new Arac { Plaka = "07NM99", MarkaModel = "Volkswagen Passat", GunlukFiyat = 12200, Sinif = "Sedan" });
            araclar.Add(new Arac { Plaka = "07Z803", MarkaModel = "Renault Megane", GunlukFiyat = 8250, Sinif = "Sedan" });
            araclar.Add(new Arac { Plaka = "27CD547", MarkaModel = "Fiat Aegea", GunlukFiyat = 7000, Sinif = "Hatchback" });
            araclar.Add(new Arac { Plaka = "01OLB5894", MarkaModel = "SKODA Scala", GunlukFiyat = 8500, Sinif = "Hatchback" });
            araclar.Add(new Arac { Plaka = "42EBT5510", MarkaModel = "Fiat Aegea", GunlukFiyat = 6250, Sinif = "Sedan" });
            araclar.Add(new Arac { Plaka = "07S7785", MarkaModel = "SKODA Kodiaq", GunlukFiyat = 20100, Sinif = "SUV" });
            araclar.Add(new Arac { Plaka = "35FUS8097", MarkaModel = "Ford Focus", GunlukFiyat = 9300, Sinif = "Sedan" });
            araclar.Add(new Arac { Plaka = "07NV462", MarkaModel = "Renault ZOE", GunlukFiyat = 5600, Sinif = "Hatchback" });
            araclar.Add(new Arac { Plaka = "42RC203", MarkaModel = "Honda CIVIC", GunlukFiyat = 6800, Sinif = "Sedan" });

            // Başlangıçta 0 gelmemesi için birkaç örnek kayıt ekledik.
            rezervasyonlar.Add(new Rezervasyon { Musteri = "Ege SÜMERLİ", Plaka = "34RENT01", Baslangic = DateTime.Now, Bitis = DateTime.Now.AddDays(2), ToplamUcret = 9000 });
            rezervasyonlar.Add(new Rezervasyon { Musteri = "Hazal GÜNDÜZ", Plaka = "35FUS8097", Baslangic = DateTime.Now, Bitis = DateTime.Now.AddDays(5), ToplamUcret = 46500 });
            rezervasyonlar.Add(new Rezervasyon { Musteri = "Meltem ALTIN", Plaka = "01OLB5894", Baslangic = DateTime.Now, Bitis = DateTime.Now.AddDays(7), ToplamUcret = 59500 });
            rezervasyonlar.Add(new Rezervasyon { Musteri = "Sezin ŞENGÖZ", Plaka = "07CNS581", Baslangic = DateTime.Now, Bitis = DateTime.Now.AddDays(4), ToplamUcret = 35200 });
            rezervasyonlar.Add(new Rezervasyon { Musteri = "Ahmet DİRENÇ", Plaka = "35VHL6846", Baslangic = DateTime.Now, Bitis = DateTime.Now.AddDays(10), ToplamUcret = 180000 });
            rezervasyonlar.Add(new Rezervasyon { Musteri = "Bayram TURANER", Plaka = "07NM99", Baslangic = DateTime.Now, Bitis = DateTime.Now.AddDays(6), ToplamUcret = 73200 });
            rezervasyonlar.Add(new Rezervasyon { Musteri = "Kazım KÖRÜKÇÜ", Plaka = "42RC203", Baslangic = DateTime.Now, Bitis = DateTime.Now.AddDays(3), ToplamUcret = 20400 });
        }

        static void MusteriGecmisi() 
        {
            Console.Write("Müşteri adı ve/veya soyadı ile arayın: ");
            string input = Console.ReadLine();
            if (string.IsNullOrWhiteSpace(input))
            {
                Console.WriteLine("Geçerli bir isim giriniz: ");
                return;
            }

            var buldu = rezervasyonlar
                .Where(r => r.Musteri != null && r.Musteri.IndexOf(input, StringComparison.OrdinalIgnoreCase) >= 0)
                .OrderByDescending(r => r.Baslangic)
                .ToList();

            if (!buldu.Any())
            {
                Console.WriteLine("Bu müşteri için kayıt bulunamadı.");
                return;
            }

            Console.WriteLine("\n{0,-25} {1,-12} {2,-12} {3,-12} {4,-12}", "Müşteri", "Plaka", "Başlangıç", "Bitiş", "Ücret");
            Console.WriteLine(new string('-', 80));
            foreach (var r in buldu)
            {
                Console.WriteLine("{0,-25} {1,-12} {2,-12} {3,-12} {4,-12:C2}",
                    r.Musteri, r.Plaka, r.Baslangic.ToShortDateString(), r.Bitis.ToShortDateString(), r.ToplamUcret);
            }

            Console.WriteLine(new string('-', 80));
            Console.WriteLine($"Toplam Kayıt: {buldu.Count}, Toplam Harcama: {buldu.Sum(x => x.ToplamUcret):C2}");

        }

        static void RezervasyonAra()
        {
            Console.WriteLine("Arama Türünü Seçiniz: 1) Müşteri  2) Plaka  3) Tarih Aralığı  4) Hepsi");
            Console.Write("Seçiminiz (1-4): ");
            string sec = Console.ReadLine();

            List<Rezervasyon> sonuc = new List<Rezervasyon>();

            switch (sec)
            {
                case "1":
                    Console.Write("Aranacak Müşterinin Adı/Soyadı: ");
                    string musteri = Console.ReadLine();
                    if (!string.IsNullOrWhiteSpace(musteri))
                    {
                        sonuc = rezervasyonlar.Where(r => r.Musteri != null && r.Musteri.IndexOf(musteri, StringComparison.OrdinalIgnoreCase) >= 0).ToList();
                    }
                    break;

                case "2":
                    Console.Write("Aranacak Plaka (Tam veya Kısmi): ");
                    string plaka = Console.ReadLine();
                    if (!string.IsNullOrWhiteSpace(plaka))
                    {
                        sonuc = rezervasyonlar.Where(r => r.Plaka != null && r.Plaka.IndexOf(plaka, StringComparison.OrdinalIgnoreCase) >= 0).ToList();
                    }
                    break;

                case "3":
                    Console.Write("Başlangıç tarihi (GG.AA.YYYY): ");
                    string basStr = Console.ReadLine();
                    Console.Write("Bitiş tarihi (GG.AA.YYYY): ");
                    string bitStr = Console.ReadLine();
                    if (DateTime.TryParse(basStr, out DateTime bas) && DateTime.TryParse(bitStr, out DateTime bit))
                    {
                        // Tarih aralığıyla çakışan rezervasyonları getir
                        sonuc = rezervasyonlar.Where(r => (bas < r.Bitis && bit > r.Baslangic)).ToList();
                    }
                    else
                    {
                        Console.WriteLine("Geçersiz tarih girdiniz.");
                        return;
                    }
                    break;

                case "4":
                    sonuc = rezervasyonlar.ToList();
                    break;
                default:
                    Console.WriteLine("Geçersiz seçim.");
                    return;

                    if (sonuc == null || sonuc.Count == 0)
                    {
                        Console.WriteLine("Arama kriterlerine uygun rezervasyon bulunamadı.");
                        return;
                    }

                    Console.WriteLine("\n{0,-25} {1,-12} {2,-12} {3,-12} {4,-12}", "Müşteri", "Plaka", "Başlangıç", "Bitiş", "Ücret");
                    Console.WriteLine(new string('-', 80));
                    foreach (var r in sonuc.OrderByDescending(x => x.Baslangic))
                    {
                        Console.WriteLine("{0,-25} {1,-12} {2,-12} {3,-12} {4,-12:C2}",
                            r.Musteri, r.Plaka, r.Baslangic.ToShortDateString(), r.Bitis.ToShortDateString(), r.ToplamUcret);
                    }

                    Console.WriteLine(new string('-', 80));
                    Console.WriteLine($"Bulunan Toplam Rezervasyon: {sonuc.Count}");
            }
        }
    }
}
