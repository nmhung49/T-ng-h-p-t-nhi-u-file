Function GetData(ByVal FileName As String, ByVal SheetName As String, ByVal RangeAddress As String, _
            ByVal HasTitle As Boolean, ByVal UseTitle As Boolean)
            
  Dim rsCon As Object, rsData As Object, cat As Object, tbl As Object
  Dim tmpArr, Arr()
  Dim szConnect As String, szSQL As String, tmp As String
  Dim lCount As Long, lR As Long, lC As Long, lVer As Long
  lVer = Val(Application.Version)
  Set rsCon = CreateObject("ADODB.Connection")
  Set rsData = CreateObject("ADODB.Recordset")
  Set cat = CreateObject("ADOX.Catalog")
  
  If lVer < 12 Then
    szConnect = "Provider=Microsoft.Jet.OLEDB.4.0;Data Source=" & FileName & ";" & _
                "Extended Properties=""Excel 8.0;HDR=" & IIf(HasTitle, "Yes", "No") & """;"
  Else
    szConnect = "Provider=Microsoft.ACE.OLEDB.12.0;Data Source=" & FileName & ";" & _
                "Extended Properties=""Excel 12.0;HDR=" & IIf(HasTitle, "Yes", "No") & """;"
  End If
  If SheetName = "" Then
    Dim Dbs  As Object, db As Object
    Set Dbs = CreateObject("DAO.DBEngine." & IIf(lVer < 12, "36", "120"))
    Set db = Dbs.OpenDatabase(FileName, False, False, "Excel 8.0;")
    tmp = db.TableDefs(0).Name
    tmp = Replace(tmp, " ", "?")
    tmp = Replace(tmp, "'", " ")
    tmp = WorksheetFunction.Trim(tmp)
    tmp = Replace(tmp, " ", "'")
    tmp = Replace(tmp, "?", " ")
    SheetName = tmp
    db.Close
    Set Dbs = Nothing: Set db = Nothing
  End If
  If Right(SheetName, 1) <> "$" Then SheetName = SheetName & "$"
  rsCon.Open szConnect
  cat.ActiveConnection = rsCon
  
  szSQL = "SELECT * FROM [" & SheetName & RangeAddress & "];"
  rsData.Open szSQL, rsCon, 0, 1, 1
  tmpArr = rsData.GetRows
  ReDim Arr(UBound(tmpArr, 2) - UseTitle, UBound(tmpArr, 1) + 1)
  If UseTitle Then
    For lC = LBound(tmpArr, 1) To UBound(tmpArr, 1)
      Arr(0, lC) = rsData.Fields(lC).Name
    Next
  End If
  For lR = LBound(tmpArr, 2) To UBound(tmpArr, 2)
    For lC = LBound(tmpArr, 1) To UBound(tmpArr, 1)
      Arr(lR - UseTitle, lC) = tmpArr(lC, lR)
    Next
  Next
  rsData.Close: Set rsData = Nothing
  rsCon.Close: Set rsCon = Nothing
  GetData = Arr
End Function

Sub Main()
  Dim vFile, FileItem, aRes, Target As Range
  Dim FileName As String, SheetName As String, RangeAddress As String
  On Error Resume Next
  vFile = Application.GetOpenFilename("Excel File, *.xls; *.xlsx; *.xlsm", , , , True)
  If TypeName(vFile) = "Variant()" Then
    SheetName = "Sheet1": RangeAddress = "A8:V10000"
    For Each FileItem In vFile
      FileName = CStr(FileItem)
      If UCase(FileName) <> UCase(ThisWorkbook.FullName) Then
        aRes = GetData(FileName, SheetName, RangeAddress, False, False)
        If IsArray(aRes) Then
          Set Target = Sheet1.Range("A60000").End(xlUp).Offset(1)
          Target.Resize(UBound(aRes, 1) + 1, UBound(aRes, 2) + 1).Value = aRes
        End If
      End If
    Next
    MsgBox "Done!"
  End If
End Sub
