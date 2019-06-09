# sqlitebrowser
# 可以打开微信(7.0.4)EnMicroMsg.db的sqlitebrowser

[Download DB Browser for SQLCipher.7z](https://github.com/lasting-yang/sqlitebrowser/raw/master/Release/DB%20Browser%20for%20SQLCipher.7z)


# sha256
```
4aabbf73aaf2a49c3f0bfe2251e93e9419b5c57ce0eb547ef0dc6b1de622855f *DB Browser for SQLCipher.7z
08fe568cba7a899b926c706db328e52dd241f402ecb40721014133fdcce909a6 *DB Browser for SQLCipher.exe
fe5f561ccd358f77801e086779b4f5b2673b2da34b9485079420a1714989cb35 *libeay32.dll
4f7795d41c0eabb93a019fd81c5f6e2a1558ae2ffb443778dfed129ed350a2c0 *Qt5Core.dll
7d734b80bc11f7ee84efc7fa02bcb458f8e1686282ae1aa0445da40fc8dff793 *Qt5Gui.dll
12edffe5be2d9832165f027d7c56a6b046b241ea0aed8e6fa3e85f8155e4a8da *Qt5Network.dll
475efe8c62dc52db46752bc2bc7b72ffd591f4c91f4d1cd7b978232273f15ebb *Qt5PrintSupport.dll
416885c5c9f3bd1fe51a2d48c2e45add07f3e9c6115c8190e21c0a34ce7c6f5d *Qt5Widgets.dll
450cae71fe3b917fca6fac71e654f7682eeff48b0bcf1e578b45aa758cf9aad3 *Qt5Xml.dll
a7681ebd8460a880ee24978fe93307ff973caeaff878a3469fb2da1693d57920 *sqlcipher.dll
```

![](./png/1.png)

![](./png/2.png)

![](./png/3.png)


基于[https://github.com/sqlitebrowser/sqlitebrowser/tree/50c1f46bdbbefaadee01b0674b9cf237eaff5052](https://github.com/sqlitebrowser/sqlitebrowser/tree/50c1f46bdbbefaadee01b0674b9cf237eaff5052)编译。

主要patch代码是PRAGMA参数的顺序。

```
diff --git a/src/sqlitedb.cpp b/src/sqlitedb.cpp                                                                                                                                   
index f7bfc2dd..166206ca 100644                                                                                                                                                    
--- a/src/sqlitedb.cpp                                                                                                                                                             
+++ b/src/sqlitedb.cpp                                                                                                                                                             
@@ -166,11 +166,15 @@ bool DBBrowserDB::open(const QString& db, bool readOnly)                                                                                                     
 #ifdef ENABLE_SQLCIPHER                                                                                                                                                           
     if(isEncrypted && cipherSettings)                                                                                                                                             
     {                                                                                                                                                                             
-        executeSQL(QString("PRAGMA key = %1").arg(cipherSettings->getPassword()), false, false);                                                                                  
-        executeSQL(QString("PRAGMA cipher_page_size = %1;").arg(cipherSettings->getPageSize()), false, false);                                                                    
-        executeSQL(QString("PRAGMA kdf_iter = %1;").arg(cipherSettings->getKdfIterations()), false, false);                                                                       
-        executeSQL(QString("PRAGMA cipher_hmac_algorithm = %1;").arg(cipherSettings->getHmacAlgorithm()), false, false);                                                          
-        executeSQL(QString("PRAGMA cipher_kdf_algorithm = %1;").arg(cipherSettings->getKdfAlgorithm()), false, false);                                                            
+        //executeSQL(QString("PRAGMA key = %1").arg(cipherSettings->getPassword()), false, false);                                                                                
+        //executeSQL(QString("PRAGMA cipher_page_size = %1;").arg(cipherSettings->getPageSize()), false, false);                                                                  
+        //executeSQL(QString("PRAGMA kdf_iter = %1;").arg(cipherSettings->getKdfIterations()), false, false);                                                                     
+        //executeSQL(QString("PRAGMA cipher_hmac_algorithm = %1;").arg(cipherSettings->getHmacAlgorithm()), false, false);                                                        
+        //executeSQL(QString("PRAGMA cipher_kdf_algorithm = %1;").arg(cipherSettings->getKdfAlgorithm()), false, false);                                                          
+               executeSQL(QString("PRAGMA cipher_default_kdf_iter = 4000;"), false, false);                                                                                       
+               executeSQL(QString("PRAGMA key = %1").arg(cipherSettings->getPassword()), false, false);                                                                           
+               executeSQL(QString("PRAGMA cipher_use_hmac = OFF;"), false, false);                                                                                                
+               executeSQL(QString("PRAGMA cipher_page_size = 1024;"), false, false);                                                                                              
     }                                                                                                                                                                             
 #endif                                                                                                                                                                            
     delete cipherSettings;                                                                                                                                                        
@@ -463,19 +467,22 @@ bool DBBrowserDB::tryEncryptionSettings(const QString& filePath, bool* encrypted                                                                             
                 cipherSettings = nullptr;                                                                                                                                         
                 return false;                                                                                                                                                     
             }                                                                                                                                                                     
-                                                                                                                                                                                  
+                                                                                                                                                                                  
+                       sqlite3_exec(dbHandle, QString("PRAGMA cipher_default_kdf_iter = 4000").toUtf8(), nullptr, nullptr, nullptr);                                              
             // Set the key                                                                                                                                                        
             sqlite3_exec(dbHandle, QString("PRAGMA key = %1").arg(cipherSettings->getPassword()).toUtf8(), nullptr, nullptr, nullptr);                                            
+                       sqlite3_exec(dbHandle, QString("PRAGMA cipher_use_hmac = OFF").toUtf8(), nullptr, nullptr, nullptr);                                                       
+                       sqlite3_exec(dbHandle, QString("PRAGMA cipher_page_size = 1024").toUtf8(), nullptr, nullptr, nullptr);                                                     
                                                                                                                                                                                   
             // Set the page size if it differs from the default value                                                                                                             
-            if(cipherSettings->getPageSize() != enc_default_page_size)                                                                                                            
-                sqlite3_exec(dbHandle, QString("PRAGMA cipher_page_size = %1;").arg(cipherSettings->getPageSize()).toUtf8(), nullptr, nullptr, nullptr);                          
-            if(cipherSettings->getKdfIterations() != enc_default_kdf_iter)                                                                                                        
-                sqlite3_exec(dbHandle, QString("PRAGMA kdf_iter = %1;").arg(cipherSettings->getKdfIterations()).toUtf8(), nullptr, nullptr, nullptr);                             
-            if(cipherSettings->getHmacAlgorithm() != enc_default_hmac_algorithm)                                                                                                  
-                sqlite3_exec(dbHandle, QString("PRAGMA cipher_hmac_algorithm = %1;").arg(cipherSettings->getHmacAlgorithm()).toUtf8(), nullptr, nullptr, nullptr);                
-            if(cipherSettings->getKdfAlgorithm() != enc_default_kdf_algorithm)                                                                                                    
-                sqlite3_exec(dbHandle, QString("PRAGMA cipher_kdf_algorithm = %1;").arg(cipherSettings->getKdfAlgorithm()).toUtf8(), nullptr, nullptr, nullptr);                  
+            //if(cipherSettings->getPageSize() != enc_default_page_size)                                                                                                          
+            //    sqlite3_exec(dbHandle, QString("PRAGMA cipher_page_size = %1;").arg(cipherSettings->getPageSize()).toUtf8(), nullptr, nullptr, nullptr);                        
+            //if(cipherSettings->getKdfIterations() != enc_default_kdf_iter)                                                                                                      
+            //    sqlite3_exec(dbHandle, QString("PRAGMA kdf_iter = %1;").arg(cipherSettings->getKdfIterations()).toUtf8(), nullptr, nullptr, nullptr);                           
+            //if(cipherSettings->getHmacAlgorithm() != enc_default_hmac_algorithm)                                                                                                
+            //    sqlite3_exec(dbHandle, QString("PRAGMA cipher_hmac_algorithm = %1;").arg(cipherSettings->getHmacAlgorithm()).toUtf8(), nullptr, nullptr, nullptr);              
+            //if(cipherSettings->getKdfAlgorithm() != enc_default_kdf_algorithm)                                                                                                  
+            //    sqlite3_exec(dbHandle, QString("PRAGMA cipher_kdf_algorithm = %1;").arg(cipherSettings->getKdfAlgorithm()).toUtf8(), nullptr, nullptr, nullptr);                
                                                                                                                                                                                   
             *encrypted = true;                                                                                                                                                    
 #else                                                                                                                                                                             
```