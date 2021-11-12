# simple-spp

1. Login
    
        public LoginForm() {
          initComponents();
          txtPassword.setEchoChar('●');
        }
        
        private void btnLoginMouseClicked(java.awt.event.MouseEvent evt) {                                      
        // TODO add your handling code here:
        
        String usrname;
        usrname = txtUsername.getText();
        new BerandaForm(usrname).setVisible(true);
        new SimpleSPPForm(usrname).setVisible(true);
        this.setVisible(false);
        }        
        
        public static void main(String args[]) {
       
        java.awt.EventQueue.invokeLater(new Runnable() {
            public void run() {
                new LoginForm().setVisible(true);
            }
        });
        }
        
2. NimHarus10Digit

        public class NimHarus10Digit extends Exception{

            @Override
            public String getMessage() {
                return "Nim yang Diinputkan Harus 10 Digit!!!"; 
            }
        }

3. NamaHarusAngkaException
    
        public class NimHarusAngkaException extends Exception{

        @Override
        public String getMessage() {
            return "Nim yang Dimasukan Harus Berupa Angka!!!"; 

        }
        }
        
4. NimHarusPositifException

        public class NimHarusPositifException extends Exception{

        @Override
        public String getMessage() {
            return "Nim yang Dimasukan Harus Positif!!!";
        }

        }
        
5. SemesterHarusValid


        public class SemesterHarusValid extends Exception{

        @Override
        public String getMessage() {
            return "Imputan Semester di luar Range Masa Studi!";
        }

        }

6. SimpleSPPForm 

        public class SimpleSPPForm extends javax.swing.JFrame {

          /**
           * Creates new form SimpleSPPForm
           */

          StringBuilder sb;
          DefaultTableModel model;
          String tabel_klik="";
          ButtonGroup bg1;
          public SimpleSPPForm() {
              initComponents();
          }
          public SimpleSPPForm(String Username) {
                  initComponents();


                  sb = new StringBuilder();

                  lblUser.setText("Selamat Datang, " +Username);


                  rbTF.setActionCommand("Teknik Informatika");
                  rbTI.setActionCommand("Teknik Industri");
                  bg1= new ButtonGroup();
                  bg1.add(rbTF);
                  bg1.add(rbTI);

                  rbTF.setSelected(true);

                  model = (DefaultTableModel) jTable1.getModel();
                  jTable1.setModel(model);
                  this.loadDataToTable();
              }
          public void loadDataToTable(){
              DefaultTableModel dm = (DefaultTableModel) jTable1.getModel();
              while (dm.getRowCount() > 0){
                  dm.removeRow(0);
              }
              try {
                  Connection conn = KoneksiDB.getKoneksi();
                  Statement s = conn.createStatement();

                  String query = "SELECT * FROM data_mahasiswa";
                  ResultSet result = s.executeQuery(query);

                  while (result.next()) {
                      Object [] obj = new Object[7];
                      obj [0] = result.getString("nama_mhs");
                      obj [1] = result.getString("nim_mhs");
                      obj [2] = 1900+ result.getDate("angkatan_msuk").getYear();
                      obj [3] = result.getString("semester");
                      obj [4] = result.getString("prodi");
                      obj [5] = result.getInt("sks_mhs");
                      obj [6] = result.getInt("id_mhs");


                      model.addRow(obj);
                  }
                  result.close();
                  s.close();
              } catch (SQLException e) {
                  System.out.println(e.getMessage());
              } finally{
                  tabel_klik="";
                  jTable1.getColumnModel().getColumn(6).setMinWidth(0);
                  jTable1.getColumnModel().getColumn(6).setMaxWidth(0);
              }
          }
          public void tampilDataMahasiswa () throws NimHarusAngkaException, NimHarusPositifException, NimHarus10Digit, SemesterHarusValid
          {
              long nim ;
              try {
                  nim = Long.parseLong(txtNIM.getText());
              } catch (NumberFormatException e) {
                  throw new NimHarusAngkaException();
              }

              if ( nim < 0) {
                  throw new NimHarusPositifException();

              }

              String Nim = txtNIM.getText();
              int intlength = Nim.length();

              if(intlength != 10){
                  throw new NimHarus10Digit();
              }

              int Semester;
              Semester = Integer.parseInt(txtSemester.getText());
              if (Semester < 1|| Semester > 14 )
              {
                  throw new SemesterHarusValid();
              } 

              taPrint.append("\n--------***DATA SPP MAHASISWA***--------");
              taPrint.append("\nNama\t\t:  " +txtNama.getText());
              taPrint.append("\nNIM\t\t:  " +txtNIM.getText());
              taPrint.append("\nAngkatan\t\t:  " +cmbAngkatan.getSelectedItem());
              taPrint.append("\nSemester\t\t:  " +txtSemester.getText());
              taPrint.append("\nProgram Studi\t\t:  " +bg1.getSelection().getActionCommand());
              taPrint.append("\nJumlah sks yang diambil\t:  " +txtSKS.getText());
          }
          public void resetData() {
              txtNama.setText("");
              txtNIM.setText("");
              cmbAngkatan.setSelectedItem("2020");
              txtSemester.setText("");
              rbTF.setSelected(true);
              txtSKS.setText("");
              taPrint.setText("");
          }
          
          private void btnLogoutMouseClicked(java.awt.event.MouseEvent evt) {                                       
              // TODO add your handling code here:
              new LoginForm().setVisible(true);
              this.setVisible(false);
          }
          
          private void bttnPrintMouseClicked(java.awt.event.MouseEvent evt) {                                       
              // TODO add your handling code here:
              try {
                  this.tampilDataMahasiswa();
              } catch (NimHarusAngkaException e) {
                  JOptionPane.showMessageDialog(this, e.getMessage());
              } catch (NimHarusPositifException e){
                  JOptionPane.showMessageDialog(this, e.getMessage());
              }catch (NimHarus10Digit e) {
                  JOptionPane.showMessageDialog(this, e.getMessage());
              }catch (SemesterHarusValid e) {
                  JOptionPane.showMessageDialog(this, e.getMessage());
              }

              try {
                  Connection conn = KoneksiDB.getKoneksi();

                  String query = "INSERT INTO data_mahasiswa" +
                          "(nama_mhs, nim_mhs, angkatan_msuk, semester, prodi, sks_mhs)" +
                          "VALUES(?,?,?,?,?,?)";
                  PreparedStatement p = conn.prepareStatement(query);

                  p.setString(1, txtNama.getText());
                  p.setString(2, txtNIM.getText());
                  p.setString(3, cmbAngkatan.getSelectedItem().toString());
                  p.setString(4, txtSemester.getText());
                  p.setString(5, bg1.getSelection().getActionCommand());
                  p.setString(6, txtSKS.getText());

                  int rowInserted = p.executeUpdate();
                  if(rowInserted > 0){
                      JOptionPane.showMessageDialog(this, "Data berhasil disimpan", "Succes", JOptionPane.INFORMATION_MESSAGE);
                  }
                  p.close();

              } catch (Exception e) {
                  JOptionPane.showMessageDialog(this, "Data gagal disimpan", "Error", JOptionPane.ERROR_MESSAGE);
              }
              finally{
                  this.loadDataToTable();
              }
          }                              
          
          
           private void bttnDeleteMouseClicked(java.awt.event.MouseEvent evt) {                                        
                // TODO add your handling code here:
                    String id_mhs = txtCoba.getText();

                try{
                    com.mysql.jdbc.Statement statement = (com.mysql.jdbc.Statement) KoneksiDB.getKoneksi().createStatement();
                    statement.executeUpdate("delete from data_mahasiswa where id_mhs = '" + id_mhs + "'");
                    JOptionPane.showMessageDialog(this,"An existing user was delete succesfully!", "Succes", JOptionPane.INFORMATION_MESSAGE);                    
                } catch (Exception e) {
                    JOptionPane.showMessageDialog(this, "Proses delete data gagal!", "Error", JOptionPane.ERROR_MESSAGE);
                }
                finally{
                    this.resetData();
                    this.loadDataToTable();
                }
            }      
            
            
            private void bttnUpdateMouseClicked(java.awt.event.MouseEvent evt) {                                        
                // TODO add your handling code here:

                    String id = txtCoba.getText();
                    String nama_mhs = txtNama.getText();
                    String nim_mhs = txtNIM.getText();
                    String angkatan_msuk = cmbAngkatan.getSelectedItem().toString();
                    String semester = txtSemester.getText();
                    String prodi = bg1.getSelection().getActionCommand();
                    String sks_mhs = txtSKS.getText();

                    try{
                        com.mysql.jdbc.Statement statement = (com.mysql.jdbc.Statement) KoneksiDB.getKoneksi().createStatement();
                        statement.executeUpdate ("update data_mahasiswa SET nama_mhs='"+nama_mhs+"'," + "nim_mhs='"+nim_mhs+"',"
                                + "angkatan_msuk='"+angkatan_msuk+"'," + "semester='"+semester+"'," + "prodi='"+prodi+"',"
                                        + "sks_mhs='"+sks_mhs+"' " + "WHERE id_mhs = '"+id+"'");
                        statement.close();
                        JOptionPane.showMessageDialog(this,"An existing user was update succesfully!", "Succes", JOptionPane.INFORMATION_MESSAGE);

                    }catch (Exception e){
                        JOptionPane.showMessageDialog(this, "Proses update data gagal!", "Error", JOptionPane.ERROR_MESSAGE);
                    }
                    finally{
                        this.resetData();
                        this.loadDataToTable();
                    }
            }          
            
            
             private void jTable1MouseClicked(java.awt.event.MouseEvent evt) {                                     
                  // TODO add your handling code here:
                  int row = jTable1.getSelectedRow();
                  tabel_klik = (jTable1.getModel().getValueAt(row, 6).toString());
                  txtCoba.setText((tabel_klik));

                  txtNama.setText(jTable1.getModel().getValueAt(row, 0).toString());
                  txtNIM.setText(jTable1.getModel().getValueAt(row, 1).toString());
                  cmbAngkatan.setSelectedItem(jTable1.getModel().getValueAt(row, 2).toString());
                  txtSemester.setText(jTable1.getModel().getValueAt(row, 3).toString());

                  String row_prodi = jTable1.getModel().getValueAt(row, 4).toString();

                  if (row_prodi.equals("Teknik Industri")) {
                      rbTI.setSelected(true);
                  }
                  else{
                      rbTF.setSelected(true);
                  }

                  txtSKS.setText(jTable1.getModel().getValueAt(row, 5).toString());

              }                   

