import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

/**
 * Sadece Java ve socket kullanan basit web sunucusu.
 * Çalıştırma:  javac WebSunucu.java  ->  java WebSunucu
 * Tarayıcıdan: http://localhost:1989
 */
public class WebSunucu {

    private static final int PORT = 1989;

    // >>> Kendi bilgilerinizle değiştirin <<<
    private static final String AD_SOYAD = "Ad Soyad";
    private static final String OGRENCI_NO = "Öğrenci No: 123456789";

    private static final String HTML =
        "<!DOCTYPE html>\n" +
        "<html lang=\"tr\">\n" +
        "<head>\n" +
        "  <meta charset=\"UTF-8\">\n" +
        "  <title>" + AD_SOYAD + "</title>\n" +
        "  <style>\n" +
        "    body { background-color: #f0f4f8; font-family: 'Segoe UI', Arial, sans-serif; margin: 40px; }\n" +
        "    h1 { color: #1a237e; font-family: 'Georgia', serif; text-align: center; }\n" +
        "    h2 { color: #c62828; font-family: 'Verdana', sans-serif; text-align: center; }\n" +
        "    .bio { background-color: #ffffff; border-left: 6px solid #1a237e; border-radius: 8px;\n" +
        "           padding: 20px; max-width: 700px; margin: 30px auto; box-shadow: 0 2px 8px rgba(0,0,0,0.15); }\n" +
        "    .bio p { color: #333333; font-size: 18px; line-height: 1.6; font-family: 'Trebuchet MS', sans-serif; }\n" +
        "    .vurgu { color: #2e7d32; font-weight: bold; }\n" +
        "    .italik { color: #6a1b9a; font-style: italic; font-family: 'Courier New', monospace; }\n" +
        "  </style>\n" +
        "</head>\n" +
        "<body>\n" +
        "  <h1>" + AD_SOYAD + "</h1>\n" +
        "  <h2>" + OGRENCI_NO + "</h2>\n" +
        "  <div class=\"bio\">\n" +
        "    <p>Merhaba! Ben <span class=\"vurgu\">" + AD_SOYAD + "</span>. " +
        "Bilgisayar programcılığı alanında eğitim görüyorum.</p>\n" +
        "    <p>Java, ağ programlama ve web teknolojilerine ilgi duyuyorum. " +
        "Boş zamanlarımda <span class=\"vurgu\">kitap okumayı</span> ve " +
        "<span class=\"vurgu\">yeni teknolojileri</span> keşfetmeyi severim.</p>\n" +
        "    <p class=\"italik\">Bu sayfa, yalnızca Java socket'leri ile yazılmış bir web sunucusundan gönderilmiştir.</p>\n" +
        "  </div>\n" +
        "</body>\n" +
        "</html>\n";

    public static void main(String[] args) throws IOException {
        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("Sunucu başlatıldı: http://localhost:" + PORT);
            while (true) {
                Socket istemci = serverSocket.accept();
                new Thread(() -> istekIsle(istemci)).start();
            }
        }
    }

    private static void istekIsle(Socket istemci) {
        try (Socket s = istemci;
             BufferedReader okuyucu = new BufferedReader(
                     new InputStreamReader(s.getInputStream(), StandardCharsets.UTF_8));
             OutputStream cikis = s.getOutputStream()) {

            // İstek satırını ve başlıkları oku (boş satıra kadar)
            String satir = okuyucu.readLine();
            if (satir == null) return;
            System.out.println("İstek: " + satir);
            while ((satir = okuyucu.readLine()) != null && !satir.isEmpty()) {
                // başlıklar yok sayılıyor
            }

            byte[] govde = HTML.getBytes(StandardCharsets.UTF_8);
            String baslik =
                "HTTP/1.1 200 OK\r\n" +
                "Content-Type: text/html; charset=UTF-8\r\n" +
                "Content-Length: " + govde.length + "\r\n" +
                "Connection: close\r\n" +
                "\r\n";

            cikis.write(baslik.getBytes(StandardCharsets.UTF_8));
            cikis.write(govde);
            cikis.flush();
        } catch (IOException e) {
            System.err.println("Bağlantı hatası: " + e.getMessage());
        }
    }
}
