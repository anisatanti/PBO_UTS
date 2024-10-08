# **UTS PBO SOAL NOMOR 6**
___
## **Berikut Langkah-langkah membuat CRUD dengan Java Swing untuk entitas MataKuliah dengan atribut: KodeMK, SKS, NamaMK, SemesterAjar.**
**1. Buat project baru** dengan nama UTS_PBO pada Netbeans namun unchecklist pada Create Main Class kemudian klik Finish.

**2. Klik kanan pada default package kemudian pilih New, klik JFrame Form lalu beri nama FrameBuku.**

**3. Buat Kelas DbUtils** dengan cara klik kanan pada default package kemudian pilih New, klik Java Class. Kelas DbUtils ini berfungsi mengonversi objek ResultSet dari database menjadi TableModel agar dapat ditampilkan pada JTable di GUI java Swing. Dengan demikian, kita dapat meneruskan data menggunakan object ResultSet ke Jtable tanpa menulis kodenya secara manual.

```
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.util.Vector;
import javax.swing.table.DefaultTableModel;
import javax.swing.table.TableModel;

public class DbUtils {

    public static TableModel resultSetToTableModel(ResultSet rs) {
        try {
            ResultSetMetaData metaData = rs.getMetaData();
            int numberOfColumns = metaData.getColumnCount();
            Vector columnNames = new Vector();

            // Get the column names
            for (int column = 0; column < numberOfColumns; column++) {
                columnNames.addElement(metaData.getColumnLabel(column + 1));
            }
            // Get all rows.
            Vector rows = new Vector();
            while (rs.next()) {
                Vector newRow = new Vector();

                for (int i = 1; i <= numberOfColumns; i++) {
                    newRow.addElement(rs.getObject(i));
                }
                rows.addElement(newRow);
            }
            return new DefaultTableModel(rows, columnNames);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}

```

**4. Membuat Database berisi tabel MataKuliah** pada PostgreSQL dengan rincian sebagai berikut. KodeMK dengan tipe data INT dan set sebagai primary key, SKS dengan tipe data INT, NamaMK dengan tipe data Varchar(50), serta SMT dengan tipe data INT.


![Screenshot 2024-10-08 125528](https://github.com/user-attachments/assets/2cb59364-0d5d-4b34-89e5-aaba6fc24731)
![Screenshot 2024-10-08 125552](https://github.com/user-attachments/assets/dc4186ff-9593-441d-864b-32970da5fc6f)

**5. Pada Project UTS_PBO di Netbeans bagian Libraries klik kanan pilih Add Library.** Setelah muncul pop up, pilih PostgreSQL JDBC Driver untuk menyambungkan database pada PostgreSQL dengan project UTS_PBO.



![Screenshot 2024-10-08 130210](https://github.com/user-attachments/assets/e4a3a42c-80b9-45bf-bd49-7e66356137c9)

![Screenshot 2024-10-08 130232](https://github.com/user-attachments/assets/2b9786c8-26b2-4846-b0ca-1fa8593a1110)

![Screenshot 2024-10-08 134620](https://github.com/user-attachments/assets/285090aa-e3a3-404a-b69e-60483f1d3940)

**6. Mulai membuat desain tampilan GUI Java Swing pada FrameMataKuliah.** Menggunakan JLable pada judul tampilan “DATA MATA KULIAH”, Kode MK, SKS, Nama MK, Semester Aja. Menggunakan JTextField untuk mengisi atau menginput dataKode MK, SKS, Nama MK, Semester Ajar. Menggunakan JButton pada tombol Insert, Update, Delete, Clear, dan Exit. Menggunakan JTable untuk menampilkan Table Data Mata Kuliah yang berisi 4 kolom yakni Kode MK, SKS, Nama MK, Semester Ajar.

![Screenshot 2024-10-08 132228](https://github.com/user-attachments/assets/fe039158-9666-4cd8-90f3-25aaa42c990a)

**7. Mulai membuat code program pada source GUI Java Swing yang telah di desain dengan rincian sebagai berikut.**

 a.) Deklarasikan beberapa import untuk menangani koneksi database, operasi SQL, dan interaksi dengan pengguna di Java. Inisialisasi nama database yang akan digunakan beserta password PostgreSQL.
 ```
package UTS;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.io.InputStreamReader;
import java.io.BufferedReader;
import javax.swing.JOptionPane;
import java.sql.SQLException;
/**
 *
 * @author LENOVO
 */
public class FrameMataKuliah extends javax.swing.JFrame {
    Connection conn;
    PreparedStatement pst;
    ResultSet rs;

    String driver = "org.postgresql.Driver";
    String koneksi = "jdbc:postgresql://localhost:5432/uts_pbo";
    String user = "postgres";
    String password = "1234";
    InputStreamReader inputStreamReader = new InputStreamReader(System.in);
    BufferedReader input = new BufferedReader(inputStreamReader);
```
b.) Membuat method koneksi untuk menampilkan koneksi database dengan GUI apakah berhasil atau error. Membuat method tampil dengan perintah sql “SELECT * FROM MataKuliah;” untuk menampilkan data yang di inputkan. Public FrameBuku merupakan constructor dari kelas FrameBuku berisi method initComponents untuk menginisialisasi komponen GUI method koneksi yang merupakan method untuk mengatur koneksi ke database, dan method tampil untuk menampilkan data ke dalam tabel dari database. serta membuat method clear untuk menghapus tulisan pada tampilan gui tetapi dieksekusi pada btnClear.
```
  public FrameMataKuliah() {
        initComponents();
        koneksi();
        tampil();
    }

    public void koneksi() {
        try {
            
            Class.forName(driver);

            conn = DriverManager.getConnection(koneksi, user, password);
            System.out.println("Koneksi berhasil!");
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("Koneksi gagal!");
        }
    }

    public void tampil() {
        try {
            String sql = "SELECT * FROM MataKuliah";
            pst = conn.prepareStatement(sql);
            rs = pst.executeQuery();

            tblMK.setModel(DbUtils.resultSetToTableModel(rs));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void clear() {
        tfKode.setText("");
        tfSks.setText("");
        tfNamaMK.setText("");
        tfSmt.setText("");
    }
```
c.) Mengisi program btnInsertActionPerformed. Btn Insert berfungsi untuk menambahkan data ke dalam database. Berisi perintah sql "INSERT INTO MataKuliah (KodeMK, SKS, NamaMK, SemesterAjar) VALUES (?, ?, ?, ?)"; Program akan berhasil Insert data atau menambahkan data jika semua bagian terisi. Jika masih ada yang belum terisi maka program akan menampilkan peringatan untuk mengisi data yang kosong untuk diisi.
```
    private void btnInsertActionPerformed(java.awt.event.ActionEvent evt) {                                          
        try {
            StringBuilder errorMessage = new StringBuilder();

            
            if (tfKode.getText().isEmpty()) {
                errorMessage.append("Kode Mata Kuliah diisi dulu yaa!\n");
            }
            if (tfSks.getText().isEmpty()) {
                errorMessage.append("SKS tidak boleh kosong!\n");
            }
            if (tfNamaMK.getText().isEmpty()) {
                errorMessage.append("Nama Mata Kuliah diisi dulu yaa!\n");
            }
            if (tfSmt.getText().isEmpty()) {
                errorMessage.append("Semester Ajar diisi dulu yaa!\n");
            }

            if (errorMessage.length() > 0) {
                JOptionPane.showMessageDialog(null, errorMessage.toString(), "Input Error", JOptionPane.WARNING_MESSAGE);
                return; 
            }
       
            int sks, semesterAjar;
            try {
                sks = Integer.parseInt(tfSks.getText());
                semesterAjar = Integer.parseInt(tfSmt.getText());
            } catch (NumberFormatException e) {
                JOptionPane.showMessageDialog(null, "SKS dan Semester Ajar harus berupa angka!", "Input Error", JOptionPane.WARNING_MESSAGE);
                return; 
            }

            String sql = "INSERT INTO MataKuliah (KodeMK, SKS, NamaMK, SemesterAjar) VALUES (?, ?, ?, ?)";
            pst = conn.prepareStatement(sql);

            pst.setString(1, tfKode.getText());
            pst.setInt(2, sks); 
            pst.setString(3, tfNamaMK.getText());
            pst.setInt(4, semesterAjar); 
            
            int rowsInserted = pst.executeUpdate();


            if (rowsInserted > 0) {
                tampil();

                JOptionPane.showMessageDialog(null, "Data Mata Kuliah berhasil ditambahkan", "Sukses", JOptionPane.INFORMATION_MESSAGE);
            }

        } catch (SQLException e) {
            JOptionPane.showMessageDialog(null, "Terjadi kesalahan: " + e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            e.printStackTrace();
        }

    }  

```
d.) Mengisi program btnUpdateActionPerformed. btnUpdate berfungsi untuk mengupdate data yang telah di input. Berisi perintah sql "UPDATE MataKuliah SET SKS=?, NamaMK=?, SemesterAjar=? WHERE KodeMK=?";. Program akan berhasil mengupdate data jika semua bagian telah terisi. Jika masih ada yang belum terisi maka program akan menampilkan peringatan untuk mengisi data yang kosong untuk diisi.
```
 private void btnUpdateActionPerformed(java.awt.event.ActionEvent evt) {                                          
        if (conn == null) {
            JOptionPane.showMessageDialog(null, "Koneksi ke database tidak tersedia.");
            return;
        }

        try {
            StringBuilder errorMessage = new StringBuilder();

          
            if (tfKode.getText().isEmpty()) {
                errorMessage.append("Kode Mata Kuliah harus diisi!\n");
            }
            if (tfSks.getText().isEmpty()) {
                errorMessage.append("SKS harus diisi!\n");
            }
            if (tfNamaMK.getText().isEmpty()) {
                errorMessage.append("Nama Mata Kuliah harus diisi!\n");
            }
            if (tfSmt.getText().isEmpty()) {
                errorMessage.append("Semester Ajar harus diisi!\n");
            }

            
            if (errorMessage.length() > 0) {
                JOptionPane.showMessageDialog(null, errorMessage.toString(), "Input Error", JOptionPane.WARNING_MESSAGE);
                return; 
            }

            
            String sql = "UPDATE MataKuliah SET SKS=?, NamaMK=?, SemesterAjar=? WHERE KodeMK=?";
            pst = conn.prepareStatement(sql);

            // Set parameter dari komponen GUI
            pst.setInt(1, Integer.parseInt(tfSks.getText())); 
            pst.setString(2, tfNamaMK.getText());
            pst.setInt(3, Integer.parseInt(tfSmt.getText())); 
            pst.setString(4, tfKode.getText());

            // Eksekusi update
            int updated = pst.executeUpdate();

            if (updated > 0) {
            
                tampil(); 
                JOptionPane.showMessageDialog(null, "Data berhasil di-update", "Sukses", JOptionPane.INFORMATION_MESSAGE);
            } else {
                JOptionPane.showMessageDialog(null, "Data gagal di-update. Periksa Kode Mata Kuliah.");
            }

        } catch (Exception e) {
            e.printStackTrace();
            JOptionPane.showMessageDialog(null, "Terjadi kesalahan: " + e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }

    }                                         

    private void btnDeleteActionPerformed(java.awt.event.ActionEvent evt) {                                          
        String id = tfKode.getText();

        if (id.isEmpty()) {
            
            JOptionPane.showMessageDialog(null, "Kode MK diisi dulu yaa!", "Input Error", JOptionPane.WARNING_MESSAGE);
            return;
        }
        int confirm = JOptionPane.showConfirmDialog(null, "Yakin ingin menghapus data dengan Kode MK: " + id + "?",
                "Konfirmasi Hapus", JOptionPane.YES_NO_OPTION);

        if (confirm == JOptionPane.YES_OPTION) {
            try {
                
                String query = "DELETE FROM MataKuliah WHERE KodeMK = ?";
                pst = conn.prepareStatement(query);
                pst.setString(1, id);

               
                int deleted = pst.executeUpdate();
               
                tampil();
                String message = (deleted > 0) ? "Data berhasil dihapus!" : "Kode MK tidak ditemukan!";
                JOptionPane.showMessageDialog(null, message);

            } catch (Exception e) {
                JOptionPane.showMessageDialog(null, "Terjadi kesalahan: " + e.getMessage(), "Error",
                        JOptionPane.ERROR_MESSAGE);
                e.printStackTrace();
            }
        }
    }                                         


```
e.) Mengisi program btnDeleteActionPerformed. btnDelete berfungsi untuk menghapus data dalam tabel database. Berisi perintah sql “DELETE FROM MataKuliah WHERE KodeMK= ?”
```
 private void btnDeleteActionPerformed(java.awt.event.ActionEvent evt) {                                          
        String id = tfKode.getText();

        if (id.isEmpty()) {
            
            JOptionPane.showMessageDialog(null, "Kode MK diisi dulu yaa!", "Input Error", JOptionPane.WARNING_MESSAGE);
            return;
        }
        int confirm = JOptionPane.showConfirmDialog(null, "Yakin ingin menghapus data dengan Kode MK: " + id + "?",
                "Konfirmasi Hapus", JOptionPane.YES_NO_OPTION);

        if (confirm == JOptionPane.YES_OPTION) {
            try {
                
                String query = "DELETE FROM MataKuliah WHERE KodeMK = ?";
                pst = conn.prepareStatement(query);
                pst.setString(1, id);

               
                int deleted = pst.executeUpdate();
               
                tampil();
                String message = (deleted > 0) ? "Data berhasil dihapus!" : "Kode MK tidak ditemukan!";
                JOptionPane.showMessageDialog(null, message);

            } catch (Exception e) {
                JOptionPane.showMessageDialog(null, "Terjadi kesalahan: " + e.getMessage(), "Error",
                        JOptionPane.ERROR_MESSAGE);
                e.printStackTrace();
            }
        }
    }                                         

```
f.) Mengisi program tblMKMouseClicked. tblBuku berfungsi untuk menampilkan data dalam bentuk tabel dan memungkinkan pengguna untuk mengeditnya.
```
 private void tblMKMouseClicked(java.awt.event.MouseEvent evt) {                                   
        int row = tblMK.getSelectedRow();

        
        String kode = tblMK.getValueAt(row, 0).toString();
        String sks = tblMK.getValueAt(row, 1).toString();
        String namaMK = tblMK.getValueAt(row, 2).toString();
        String smt = tblMK.getValueAt(row, 3).toString();

      
        tfKode.setText(kode);
        tfSks.setText(sks);
        tfNamaMK.setText(namaMK);
        tfSmt.setText(smt);
    
    }                   
```
g.) Mengisi program btnClearActionPerformed. btnClear berfungsi menghapus inputan yang dimasukkan dalam Frame GUI secara keseluruhan. code btnClear sudah diinisialisasi di awal program dibawah method tampil() 
```
 private void btnClearActionPerformed(java.awt.event.ActionEvent evt) {                                         
        // TODO add your handling code here:
   clear();
```
h.) Mengisi program btnExitActionPerformed. btnExit berfungsi untuk menutup atau keluar dari aplikasi ketika pengguna mengklik tombol tersebut.
```
private void btnExitActionPerformed(java.awt.event.ActionEvent evt) {                                        
        // TODO add your handling code here:
        System.exit(0);
    }
```
**8. Hasil Eksekusi Insert data**

![Screenshot 2024-10-08 132400](https://github.com/user-attachments/assets/62c61ef1-6599-4019-814c-f3c6c484021c)

**9. Hasil Eksekusi Update data**

- Sebelum di Update
  
![Screenshot 2024-10-08 132819](https://github.com/user-attachments/assets/4594633e-25b1-4a16-aa8f-75df8a1ee7b0)
  
- Setelah di Update
  
![Screenshot 2024-10-08 132856](https://github.com/user-attachments/assets/7b1aec45-2224-44b3-8551-bf010ebecfd5)

**10. Hasil Eksekusi Delete data**

![Screenshot 2024-10-08 132955](https://github.com/user-attachments/assets/97df0616-1aee-4a3b-9492-c346d87965a2)

- Data berhasil di hapus
  
![Screenshot 2024-10-08 133018](https://github.com/user-attachments/assets/33dea6ae-468c-4f1c-a5f8-009fe17383a1)

![Screenshot 2024-10-08 133030](https://github.com/user-attachments/assets/5263d00c-f68a-4d88-aa90-a1f2b0c69789)

**11. Hasil Eksekusi Clear Data**

- Sebelum Clear data
- 
![Screenshot 2024-10-08 133154](https://github.com/user-attachments/assets/69fb4635-0497-4b6a-aec6-f9fbf0c8ae88)

- Setelah klik btnClear

![Screenshot 2024-10-08 133203](https://github.com/user-attachments/assets/9ecfc048-1e3c-4593-aa84-e3e1f870e856)



