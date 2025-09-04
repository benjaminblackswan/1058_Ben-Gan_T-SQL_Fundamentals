## How to restore a database from a .bak file (GUI version)

Let's say I want to restore the database 1058 to an earlier version.

1. Right click on the Databases folder and click on **Restore Databases**.

   <img width="364" height="178" alt="image" src="https://github.com/user-attachments/assets/474e41ad-d6b9-4984-927e-7e235cd829dc" />

2. In the **General** tab, click on Device and click on the ... button. Then click on the **Add** button.
   
   <img width="806" height="585" alt="image" src="https://github.com/user-attachments/assets/6832cc5f-64d2-4b03-987f-c5d7e106dedd" />

3. Then navigate to `C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup`, choose the .bak file you wish to restore and click OK.

   <img width="639" height="457" alt="image" src="https://github.com/user-attachments/assets/c8a17601-c893-484b-97bc-cd87f9b125d6" />

4. You can see the .bak file in the Source section, and **1058** as the database name.

   <img width="1003" height="587" alt="image" src="https://github.com/user-attachments/assets/d8ec6e8f-6171-4291-b98d-0d5dc2cf52a0" />

5. Then go to the **Files** tab, and make sure the **mdf** and **ldf** file name to be restored matches the database name. If it doesn't changes so that it matches the name of the database.
   For example, in this case, the mdf and ldf files are named TSQLV4, these were carried from the .bak file, we need to change it manually so that the Data folder gets the correct name.

   <img width="1005" height="582" alt="image" src="https://github.com/user-attachments/assets/8d9edf59-53c2-409f-bf5b-4a237d66ac8e" />

   <img width="994" height="581" alt="image" src="https://github.com/user-attachments/assets/4d0c5072-4b68-4a58-bb91-94a0f971ea7a" />
   
7. In the **Options** tab, check `WITH REPLACE` option.

   <img width="999" height="587" alt="image" src="https://github.com/user-attachments/assets/5efe957d-ff30-4984-91e0-05b08ad22cf9" />
   
8. Click OK to start the restore process.

   <img width="296" height="154" alt="image" src="https://github.com/user-attachments/assets/a2901386-12ba-48a7-bb70-e7c91dcb91f7" />
  
### Post upgrade cleaning up

1. Go to the database property and change the Logical name.

   <img width="704" height="655" alt="image" src="https://github.com/user-attachments/assets/587b0719-b48f-49b8-bac6-1daa7b127220" />





















## Stuck on Restoring...

If you see this

<img width="274" height="106" alt="image" src="https://github.com/user-attachments/assets/e0dfa42c-4341-4db5-b313-f2249edc02fa" />
