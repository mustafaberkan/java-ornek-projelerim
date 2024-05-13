import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;

public class as extends JFrame {
    private ArrayList<Randevu> randevular = new ArrayList<>();
    private ArrayList<String> hastaIsimleri = new ArrayList<>();
    private ArrayList<String> doktorIsimleri = new ArrayList<>();

    private JTextField hastaAdiField, doktorAdiField, tarihField, saatField, nedenField;
    private JTextArea randevuTextArea;

    public as() {
        setTitle("Randevu Uygulaması");
        setSize(500, 400);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLayout(new BorderLayout());

        JPanel panel = new JPanel(new GridLayout(6, 2));

        JLabel hastaAdiLabel = new JLabel("Hasta Adı:");
        hastaAdiField = new JTextField();
        panel.add(hastaAdiLabel);
        panel.add(hastaAdiField);

        JLabel doktorAdiLabel = new JLabel("Doktor Adı:");
        doktorAdiField = new JTextField();
        panel.add(doktorAdiLabel);
        panel.add(doktorAdiField);

        JLabel tarihLabel = new JLabel("Randevu Tarihi (gg.aa.yyyy):");
        tarihField = new JTextField();
        panel.add(tarihLabel);
        panel.add(tarihField);

        JLabel saatLabel = new JLabel("Randevu Saati (ss:dd):");
        saatField = new JTextField();
        panel.add(saatLabel);
        panel.add(saatField);

        JLabel nedenLabel = new JLabel("Randevu Nedeni:");
        nedenField = new JTextField();
        panel.add(nedenLabel);
        panel.add(nedenField);

        JButton randevuAlButton = new JButton("Randevu Al");
        randevuAlButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                randevuAl();
            }
        });
        panel.add(randevuAlButton);

        JButton randevulariGosterButton = new JButton("Randevuları Göster");
        randevulariGosterButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                randevulariGoster();
            }
        });
        panel.add(randevulariGosterButton);

        JButton randevuIptalButton = new JButton("Randevu İptal Et");
        randevuIptalButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                randevuIptal();
            }
        });
        panel.add(randevuIptalButton);

        add(panel, BorderLayout.NORTH);

        randevuTextArea = new JTextArea();
        add(new JScrollPane(randevuTextArea), BorderLayout.CENTER);
    }

    private void randevuAl() {
        String hastaAdi = hastaAdiField.getText();
        String doktorAdi = doktorAdiField.getText();
        String tarihStr = tarihField.getText();
        String saatStr = saatField.getText();
        String neden = nedenField.getText();

        if (hastaAdi.isEmpty() || doktorAdi.isEmpty() || tarihStr.isEmpty() || saatStr.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Lütfen tüm alanları doldurun.", "Hata", JOptionPane.ERROR_MESSAGE);
            return;
        }

        LocalDate tarih;
        LocalTime saat;
        try {
            tarih = LocalDate.parse(tarihStr, DateTimeFormatter.ofPattern("dd.MM.yyyy"));
            saat = LocalTime.parse(saatStr, DateTimeFormatter.ofPattern("HH:mm"));
        } catch (Exception e) {
            JOptionPane.showMessageDialog(this, "Geçersiz tarih veya saat formatı.", "Hata", JOptionPane.ERROR_MESSAGE);
            return;
        }

        if (tarih.isBefore(LocalDate.now())) {
            JOptionPane.showMessageDialog(this, "Geçmiş tarihler için randevu alınamaz.", "Hata", JOptionPane.ERROR_MESSAGE);
            return;
        }

        if (!hastaIsimleri.contains(hastaAdi)) {
            hastaIsimleri.add(hastaAdi);
        }
        if (!doktorIsimleri.contains(doktorAdi)) {
            doktorIsimleri.add(doktorAdi);
        }
        Randevu randevu = new Randevu(hastaAdi, doktorAdi, tarih, saat, neden);
        randevular.add(randevu);
        randevuTextArea.setText("Randevu başarıyla alındı.\n" + randevuToString(randevu));
    }

    private void randevulariGoster() {
        if (randevular.isEmpty()) {
            randevuTextArea.setText("Hiç randevu bulunmamaktadır.");
            return;
        }
        StringBuilder sb = new StringBuilder("Randevular:\n");
        for (Randevu randevu : randevular) {
            sb.append(randevuToString(randevu)).append("\n");
        }
        randevuTextArea.setText(sb.toString());
    }

    private void randevuIptal() {
        String hastaAdi = JOptionPane.showInputDialog(this, "İptal etmek istediğiniz hastanın adını girin:");
        if (hastaAdi == null || hastaAdi.isEmpty()) {
            return;
        }

        ArrayList<Randevu> iptalEdilecekRandevular = new ArrayList<>();
        for (Randevu randevu : randevular) {
            if (randevu.getHastaAdi().equals(hastaAdi)) {
                iptalEdilecekRandevular.add(randevu);
            }
        }
        if (iptalEdilecekRandevular.isEmpty()) {
            JOptionPane.showMessageDialog(this, "Belirtilen hastaya ait randevu bulunamadı.", "Hata", JOptionPane.ERROR_MESSAGE);
            return;
        }

        StringBuilder sb = new StringBuilder("İptal etmek istediğiniz randevular:\n");
        for (int i = 0; i < iptalEdilecekRandevular.size(); i++) {
            sb.append((i + 1)).append(". ").append(randevuToString(iptalEdilecekRandevular.get(i))).append("\n");
        }
        int secim = JOptionPane.showConfirmDialog(this, sb.toString() + "\nBu randevuyu iptal etmek istediğinizden emin misiniz?", "Randevu İptal", JOptionPane.YES_NO_OPTION);
        if (secim == JOptionPane.YES_OPTION) {
            for (Randevu randevu : iptalEdilecekRandevular) {
                randevular.remove(randevu);
            }
            randevuTextArea.setText("Randevu başarıyla iptal edildi.");
        }
    }

    private String randevuToString(Randevu randevu) {
        return "Hasta: " + randevu.getHastaAdi() +
                ", Doktor: " + randevu.getDoktorAdi() +
                ", Tarih: " + randevu.getTarih().format(DateTimeFormatter.ofPattern("dd.MM.yyyy")) +
                ", Saat: " + randevu.getSaat().format(DateTimeFormatter.ofPattern("HH:mm")) +
                ", Neden: " + randevu.getRandevuNedeni();
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            @Override
            public void run() {
                new as().setVisible(true);
            }
        });
    }
}
