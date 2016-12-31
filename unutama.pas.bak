unit unUtama;

{$mode delphi}

interface

uses
  Classes, SysUtils, FileUtil, Forms, Controls, Graphics, Dialogs, StdCtrls,
  ComCtrls,  CheckLst, EditBtn, ExtCtrls, fpjson, jsonparser,
  fphttpclient, Types;

{ Tfrm_utama }
const
   PICTURE_URL='http://cdn.mangaeden.com/mangasimg/';
   CHAPTER_URL='http://www.mangaeden.com/api/chapter/';
   MANGA_URL='http://www.mangaeden.com/api/manga/';
   LIST_URL='http://www.mangaeden.com/api/list/0/';

type
  Tfrm_utama = class(TForm)
    Button1: TButton;
    button2: TButton;
    Button3: TButton;
    Button4: TButton;
    clb_title: TCheckListBox;
    DirectoryEdit1: TDirectoryEdit;
    Edit1: TEdit;
    Label1: TLabel;
    Label2: TLabel;
    Label3: TLabel;
    lbl_ket: TLabel;
    lsb_manga: TListBox;
    ProgressBar1: TProgressBar;
    ProgressBar2: TProgressBar;
    ProgressBar3: TProgressBar;
    procedure Button1Click(Sender: TObject);
    procedure button2Click(Sender: TObject);
    procedure Button3Click(Sender: TObject);
    procedure Button4Click(Sender: TObject);
    procedure FormCreate(Sender: TObject);
    procedure FormDestroy(Sender: TObject);
    procedure GetMangaList;
    procedure GetChapterList(MangaCode:String);
    function GetPictureList(ChapterCode:String) : TStringList;
    procedure Image1Click(Sender: TObject);
    procedure ProgressBar3ContextPopup(Sender: TObject; MousePos: TPoint;
      var Handled: Boolean);

  private
    { private declarations }
  public

    { public declarations }
  end;

var
  frm_utama: Tfrm_utama;
  JData : TJSONData;
  MangaList:TStringList;
  ChapterList:TStringList;


implementation

{$R *.lfm}

{ Tfrm_utama }



procedure Tfrm_utama.FormCreate(Sender: TObject);
var i:integer;
begin
  MangaList:=TStringList.Create;
  ChapterList:=TStringList.Create;
  if FileExists(ExtractFilePath(Application.ExeName)+'mangalist.txt') then
  MangaList.LoadFromFile( ExtractFilePath(Application.ExeName)+'mangalist.txt' ) ;
  For i:= 0 to MangaList.Count-1  do
      lsb_manga.Items.AddText(  MangaList.Names[i]);

end;

procedure Tfrm_utama.FormDestroy(Sender: TObject);
begin
    MangaList.Free;
    ChapterList.Free;
end;

procedure Tfrm_utama.GetMangaList;
var i : integer;
    ttl,cd : string;
begin
  try
    lsb_manga.Items.clear;
    JData := GetJSON(Tfphttpclient.simpleget(LIST_URL));
    ProgressBar1.Max:=JData.FindPath('manga').Count;
    for i:= 0 to JData.FindPath('manga').Count-1 do
    begin

      ttl:=JData.FindPath('manga').Items[i].FindPath('t').AsString;
      cd:=JData.FindPath('manga').Items[i].FindPath('i').AsString;
      MangaList.Values[ttl]:=cd;
      ProgressBar1.StepBy(1);
      Application.ProcessMessages;
    end;
    MangaList.Sort;
    For i:= 0 to MangaList.Count-1 do
      lsb_manga.Items.AddText(  MangaList.Names[i]);

    MangaList.SaveToFile(ExtractFilePath(Application.ExeName)+'mangalist.txt')

  except
    raise Exception.Create('Failed To Populate Manga List');
  end;


end;

procedure Tfrm_utama.GetChapterList(MangaCode:String);
var i : integer;
    ttl,cd : string;

begin
  try
    clb_title.Clear;
    JData := GetJSON(Tfphttpclient.simpleget(MANGA_URL+MangaCode+'/'));
    ProgressBar2.Max:=JData.FindPath('chapters').Count;
    for i:= 0 to JData.FindPath('chapters').Count-1 do
    begin
      if JData.FindPath('chapters').Items[i].Items[2].IsNull then
         ttl:=JData.FindPath('chapters').Items[i].Items[0].AsString
      else ttl:=JData.FindPath('chapters').Items[i].Items[2].AsString;
      cd:=JData.FindPath('chapters').Items[i].Items[3].AsString;
      ChapterList.Values[ttl]:=cd;
      clb_title.Items.Add(ttl);

      ProgressBar2.StepBy(1);
      Application.ProcessMessages;
    end;

  except
    Exception.Create('Failed to Populate Chapter List!!');
  end;




end;

function Tfrm_utama.GetPictureList(ChapterCode: String): TStringList;
var i : integer;

begin
  Result:=TStringList.Create;
  JData := GetJSON(Tfphttpclient.simpleget(CHAPTER_URL+ChapterCode+'/'));
  ProgressBar3.Max:=JData.FindPath('images').Count;
  for i:= 0 to JData.FindPath('images').Count-1 do
  begin
    if not JData.FindPath('images').Items[i].Items[1].IsNull then
       Result.Add(JData.FindPath('images').Items[i].Items[1].AsString);

    Application.ProcessMessages;
  end;



end;

procedure Tfrm_utama.Image1Click(Sender: TObject);
begin
  end;

procedure Tfrm_utama.ProgressBar3ContextPopup(Sender: TObject;
  MousePos: TPoint; var Handled: Boolean);
begin

end;



procedure Tfrm_utama.Button1Click(Sender: TObject);
begin
      GetMangaList;
end;

procedure Tfrm_utama.button2Click(Sender: TObject);
begin
   GetChapterList(MangaList.ValueFromIndex[lsb_manga.ItemIndex]);
end;

procedure Tfrm_utama.Button3Click(Sender: TObject);
var i,j:integer;
    imagelist:TStringList;
    Stream :TMemoryStream;
    DirName,PIC_URL:String;
begin
  Stream:=TMemoryStream.Create;

  try
  for i:=0 to clb_title.Items.Count-1 do
  begin
    if clb_title.Checked[i] then
    begin
      imagelist:=GetPictureList(ChapterList.ValueFromIndex[i]);
      ProgressBar3.Max:=imagelist.Count;
      ProgressBar3.Position:=0;
      lbl_ket.Caption:='Downloading chapter '+clb_title.Items[i]+' ....';
      Application.ProcessMessages;
      For j:=imagelist.Count-1 downto 0 do //chapter list disimpan terbalik
      begin
        DirName:=DirectoryEdit1.Text+'/'+clb_title.Items[i]+'/';
        if not DirectoryExists(DirName) then
           CreateDir(DirName);
        PIC_URL:=PICTURE_URL+imagelist[j];
        TFPHTTPClient.SimpleGet(PIC_URL,stream);
        Stream.Position:=0;
        Stream.SaveToFile(DirName+IntToStr(imagelist.count-j)+ExtractFileExt(imagelist[j]));
        ProgressBar3.StepBy(1);
        Application.ProcessMessages;
        Stream.Clear;
      end;


    end;

  end;
    imagelist.Free;
    Stream.Free;

  except
    imagelist.Free;
    Stream.Free;
    Exception.Create('Failed to Download Chapter');


  end;

end;

procedure Tfrm_utama.Button4Click(Sender: TObject);
begin
   lsb_manga.ItemIndex:=  lsb_manga.Items.IndexOf(Edit1.Text);
end;

end.

