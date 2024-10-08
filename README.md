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

**5. Pada Project UTS_PBO di Netbeans bagian Libraries klik kanan pilih Add Library.** Setelah muncul pop up, pilih PostgreSQL JDBC Driver untuk menyambungkan database pada PostgreSQL dengan project SimulasiUTSPBO_Buku.



![Screenshot 2024-10-08 130210](https://github.com/user-attachments/assets/e4a3a42c-80b9-45bf-bd49-7e66356137c9)
![Screenshot 2024-10-08 130232](https://github.com/user-attachments/assets/2b9786c8-26b2-4846-b0ca-1fa8593a1110)

![Screenshot 2024-10-08 134620](https://github.com/user-attachments/assets/285090aa-e3a3-404a-b69e-60483f1d3940)

**6. Mulai membuat desain tampilan GUI Java Swing pada FrameMataKuliah.** Menggunakan JLable pada judul tampilan “DATA MATA KULIAH”, Kode MK, SKS, Nama MK, Semester Aja. Menggunakan JTextField untuk mengisi atau menginput dataKode MK, SKS, Nama MK, Semester Ajar. Menggunakan JButton pada tombol Insert, Update, Delete, Clear, dan Exit. Menggunakan JTable untuk menampilkan Table Data Mata Kuliah yang berisi 4 kolom yakni Kode MK, SKS, Nama MK, Semester Ajar.
![Screenshot 2024-10-01 224325](https://github.com/user-attachments/assets/1a58dd31-693e-4bc1-980d-023615d8799d)

**7. Mulai membuat code program pada source GUI Java Swing yang telah di desain dengan rincian sebagai berikut.**

 a.) Deklarasikan beberapa import untuk menangani koneksi database, operasi SQL, dan interaksi dengan pengguna di Java. Inisialisasi nama database yang akan digunakan beserta password PostgreSQL.
 ```
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
public class FrameBuku extends javax.swing.JFrame {
    Connection conn;
    PreparedStatement pst;
    ResultSet rs;

    String driver = "org.postgresql.Driver";
    String koneksi = "jdbc:postgresql://localhost:5432/Buku";
    String user = "postgres";
    String password = "1234";
    InputStreamReader inputStreamReader = new InputStreamReader(System.in);
    BufferedReader input = new BufferedReader(inputStreamReader);
```
b.) Membuat method koneksi untuk menampilkan koneksi database dengan GUI apakah berhasil atau error. Membuat method tampil dengan perintah sql “SELECT * FROM Buku;” untuk menampilkan data yang di inputkan. Public FrameBuku merupakan constructor dari kelas FrameBuku berisi method initComponents untuk menginisialisasi komponen GUI method koneksi yang merupakan method untuk mengatur koneksi ke database, dan method tampil untuk menampilkan data ke dalam tabel dari database.
```
        public FrameBuku() {
        initComponents();
        koneksi();
        tampil();
    }
    
    public void koneksi() {
    try {
        // Memuat driver PostgreSQL
        Class.forName(driver);
        
        // Membuat koneksi ke database
        conn = DriverManager.getConnection(koneksi, user, password);
        System.out.println("Koneksi berhasil!");
    } catch (Exception e) {
        e.printStackTrace();
        System.out.println("Koneksi gagal!");
    }
}
    public void tampil() {
    try {
        String sql = "SELECT * FROM Buku";
        pst = conn.prepareStatement(sql); 
        rs = pst.executeQuery();
        
        tblBuku.setModel(DbUtils.resultSetToTableModel(rs)); 
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
c.) Mengisi program btnInsertActionPerformed. Btn Insert berfungsi untuk menambahkan data ke dalam database. Berisi perintah sql "INSERT INTO Buku (ISBN, Judul_Buku, , Tahun_Terbit, Penerbit) VALUES (?, ?, ?, ?)"; Program akan berhasil Insert data atau menambahkan data jika semua bagian terisi. Jika masih ada yang belum terisi maka program akan menampilkan peringatan untuk mengisi data yang kosong untuk diisi.
```
private void btnInsertActionPerformed(java.awt.event.ActionEvent evt) {
     // TODO add your handling code here:
 try {
    // Variabel untuk menyimpan pesan error jika ada kolom yang kosong
    StringBuilder errorMessage = new StringBuilder();

    // Validasi setiap kolom input
    if (tfISBN.getText().isEmpty()) {
        errorMessage.append("ISBN diisi dulu yaa!\n");
    }
    if (tfJudul.getText().isEmpty()) {
        errorMessage.append("Judul Buku tidak boleh kosong!\n");
    }
    if (tfThnTerbit.getText().isEmpty()) {
        errorMessage.append("Tahun Terbit diisi dulu!\n");
    }
    if (tfPenerbit.getText().isEmpty()) {
        errorMessage.append("Penerbit diisi dulu!\n");
    }

    // Jika ada pesan error, tampilkan pop-up dan hentikan proses
    if (errorMessage.length() > 0) {
        JOptionPane.showMessageDialog(null, errorMessage.toString(), "Input Error", JOptionPane.WARNING_MESSAGE);
        return; // Hentikan proses jika ada error
    }

    // Konversi tahun terbit menjadi tipe integer
    int tahunTerbit;
    try {
        tahunTerbit = Integer.parseInt(tfThnTerbit.getText());
    } catch (NumberFormatException e) {
        JOptionPane.showMessageDialog(null, "Tahun Terbit harus berupa angka!", "Input Error", JOptionPane.WARNING_MESSAGE);
        return; // Hentikan proses jika input tidak valid
    }

    // Jika semua input sudah terisi dan valid, lanjutkan proses insert ke database
    String sql = "INSERT INTO Buku (ISBN, Judul_Buku, Tahun_Terbit, Penerbit) VALUES (?, ?, ?, ?)";
    pst = conn.prepareStatement(sql);

    pst.setString(1, tfISBN.getText());
    pst.setString(2, tfJudul.getText());
    pst.setInt(3, tahunTerbit); // Set tahun terbit sebagai integer
    pst.setString(4, tfPenerbit.getText());

    // Eksekusi query
    int rowsInserted = pst.executeUpdate();

    // Cek apakah data berhasil ditambahkan
    if (rowsInserted > 0) {
        // Refresh data di tabel terlebih dahulu
        tampil();
        
        // Setelah tabel diperbarui, tampilkan pop-up pesan sukses
        JOptionPane.showMessageDialog(null, "Data berhasil ditambahkan", "Sukses", JOptionPane.INFORMATION_MESSAGE);
    }

} catch (SQLException e) {
    JOptionPane.showMessageDialog(null, "Terjadi kesalahan: " + e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
    e.printStackTrace();
}
}
```
d.) Mengisi program btnUpdateActionPerformed. btnUpdate berfungsi untuk mengupdate data yang telah di input. Berisi perintah sql "UPDATE Buku SET Judul_Buku=?, Tahun_Terbit=?, Penerbit=? WHERE ISBN=?". Program akan berhasil mengupdate data jika semua bagian telah terisi. Jika masih ada yang belum terisi maka program akan menampilkan peringatan untuk mengisi data yang kosong untuk diisi.
```
 private void btnUpdateActionPerformed(java.awt.event.ActionEvent evt) {                                          
        // TODO add your handling code here:
    if (conn == null) {
    JOptionPane.showMessageDialog(null, "Koneksi ke database tidak tersedia.");
    return;
}
try {
    StringBuilder errorMessage = new StringBuilder();

    // Validasi setiap kolom input
    if (tfISBN.getText().isEmpty()) {
        errorMessage.append("ISBN diisi dulu yaa!\n");
    }
    if (tfJudul.getText().isEmpty()) {
        errorMessage.append("Judul Buku tidak boleh kosong!\n");
    }
    if (tfThnTerbit.getText().isEmpty()) {
        errorMessage.append("Tahun Terbit diisi dulu!\n");
    }
    if (tfPenerbit.getText().isEmpty()) {
        errorMessage.append("Penerbit diisi dulu!\n");
    }

    // Jika ada pesan error, tampilkan pop-up dan hentikan proses
    if (errorMessage.length() > 0) {
        JOptionPane.showMessageDialog(null, errorMessage.toString(), "Input Error", JOptionPane.WARNING_MESSAGE);
        return; // Hentikan proses jika ada error
    }

    // SQL query untuk update data
    String sql = "UPDATE Buku SET Judul_Buku=?, Tahun_Terbit=?, Penerbit=? WHERE ISBN=?";
    pst = conn.prepareStatement(sql);

    // Set parameter dari komponen GUI
    pst.setString(1, tfJudul.getText());
    pst.setInt(2, Integer.parseInt(tfThnTerbit.getText())); // Konversi input tahun terbit ke integer
    pst.setString(3, tfPenerbit.getText());
    pst.setString(4, tfISBN.getText());

    // Eksekusi update
    int updated = pst.executeUpdate();
    // Tampilkan pesan jika update berhasil
    if (updated > 0) {
        // Refresh data di tabel terlebih dahulu
        tampil();
        // Setelah tabel diperbarui, tampilkan pop-up pesan sukses
        JOptionPane.showMessageDialog(null, "Data berhasil di-update", "Sukses", JOptionPane.INFORMATION_MESSAGE);
    } else {
        JOptionPane.showMessageDialog(null, "Data gagal di-update. Periksa ISBN");
    }

} catch (Exception e) {
    e.printStackTrace();
    JOptionPane.showMessageDialog(null, "Terjadi kesalahan: " + e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
}

    }
```
e.) Mengisi program btnDeleteActionPerformed. btnDelete berfungsi untuk menghapus data dalam tabel database. Berisi perintah sql “DELETE FROM Buku WHERE ISBN= ?”
```
private void btnDeleteActionPerformed(java.awt.event.ActionEvent evt) {                                          
        // TODO add your handling code here:
    String id = tfISBN.getText();

    if (id.isEmpty()) {
    // Validasi jika ID Paspor kosong
     JOptionPane.showMessageDialog(null, "ISBN diisi dulu yaa!", "Input Error", JOptionPane.WARNING_MESSAGE);
        return;
    }
    int confirm = JOptionPane.showConfirmDialog(null, "Yakin ingin menghapus data dengan ISBN: " + id + "?",
            "Konfirmasi Hapus", JOptionPane.YES_NO_OPTION);

    if (confirm == JOptionPane.YES_OPTION) {
        try {
            // SQL query untuk menghapus data
            String query = "DELETE FROM Buku WHERE ISBN = ?";
             pst = conn.prepareStatement(query);
            pst.setString(1, id);

            // Eksekusi penghapusan
            int deleted = pst.executeUpdate();
            // Refresh tabel dan tampilkan pesan
            tampil();
            String message = (deleted > 0) ? "Data berhasil dihapus!" : "ISBN tidak ditemukan!";
            JOptionPane.showMessageDialog(null, message);

        } catch (Exception e) {
            JOptionPane.showMessageDialog(null, "Terjadi kesalahan: " + e.getMessage(), "Error", 
                    JOptionPane.ERROR_MESSAGE);
            e.printStackTrace();
        }
    }
    }
```
f.) Mengisi program tblBukuMouseClicked. tblBuku berfungsi untuk menampilkan data dalam bentuk tabel dan memungkinkan pengguna untuk mengeditnya.
```
private void tblBukuMouseClicked(java.awt.event.MouseEvent evt) {                                     
        // TODO add your handling code here:
     int row = tblBuku.getSelectedRow();

    // Mengambil data dari tabel sesuai dengan indeks kolom
    String id = tblBuku.getValueAt(row, 0).toString(); 
    String Judul = tblBuku.getValueAt(row, 1).toString(); 
    String Tahun = tblBuku.getValueAt(row, 2).toString(); 
    String Penerbit = tblBuku.getValueAt(row, 3).toString(); 

    // Mengisi form dengan data yang diperoleh
    tfISBN.setText(id);
    tfJudul.setText(Judul);
    tfThnTerbit.setText(Tahun);
    tfPenerbit.setText(Penerbit);

    }
```
g.) Mengisi program btnClearActionPerformed. btnClear berfungsi menghapus inputan yang dimasukkan dalam Frame GUI secara keseluruhan. 
```
 private void btnClearActionPerformed(java.awt.event.ActionEvent evt) {                                         
        // TODO add your handling code here:
   bersih(); }
   public void bersih() {
    tfISBN.setText("");
    tfJudul.setText("");
    tfThnTerbit.setText(""); 
    tfPenerbit.setText("");
    }
```
h.) Mengisi program btnExitActionPerformed. btnExit berfungsi untuk menutup atau keluar dari aplikasi ketika pengguna mengklik tombol tersebut.
```
private void btnExitActionPerformed(java.awt.event.ActionEvent evt) {                                        
        // TODO add your handling code here:
        System.exit(0);
    }
```
**8. Hasil Eksekusi Insert data**

![Screenshot 2024-10-01 220635](https://github.com/user-attachments/assets/f82a7f28-b68e-4e5a-9e2a-b33e992cef59)

**9. Hasil Eksekusi Update data**

- Sebelum di Update
  
  ![Screenshot 2024-10-01 221653](https://github.com/user-attachments/assets/fe863e40-d0b2-4d13-be15-07fdc3b9b755)
  
- Setelah di Update
  
  ![Screenshot 2024-10-01 221802](https://github.com/user-attachments/assets/d5ce0dbe-d27b-4a2a-a49c-2bca4742f43f)

**10. Hasil Eksekusi Delete data**

![Screenshot 2024-10-01 230234](https://github.com/user-attachments/assets/5f70850e-abfd-413b-b780-f3d1693f7f3d)

- Data berhasil di hapus
  
![Screenshot 2024-10-01 230257](https://github.com/user-attachments/assets/1a36202e-7ed2-4da7-948d-4a01c9a1be4b)

**11. Hasil Eksekusi Clear Data**

- Sebelum Clear data

![Screenshot 2024-10-01 230312](https://github.com/user-attachments/assets/ec83ac43-d8ed-4850-9640-242dcaddd817)

- Setelah klik btnClear

![Screenshot 2024-10-01 230323](https://github.com/user-attachments/assets/8bc424c7-d4b6-4f63-857e-de3249387635)



