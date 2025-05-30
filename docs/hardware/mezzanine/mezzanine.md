# Mezzanine cards

ここでAMANEQ用に開発されたメザニンカードについてのみ紹介します。HUL用に開発されたものはHULのUser Guide等を参照してください。

## Clock-Data-Distributer Optical (CDD-OPT)

- 製造: 有限会社ジー・エヌ・ディー
- 型番: GN-2436-1

![CDD-OPT](cdd-opt.png "Mezzanine CDD-OPT"){: #CDD-OPT width="80%"}

Mezzanine CDD-OPTは2スロット幅の大型メザニンカードです。16ch分のMIKUMARIポートを有しており、変調クロック信号を分配するために用います。ポート番号は[図](#CDD-OPT)に示されている順番でふられています。Mikumari Clock RootやMikumari Clock Hubで用います。16ポート全てにSFPモジュールを挿入した際の消費電力は、およそ10Wです。
1000BASE-SXかLX用の光ファイバーモジュールを使用してください。LANケーブル用のモジュールは使用できません。

## Mini-mezzanine Clock-Receiver (CRV)

- 製造: 有限会社ジー・エヌ・ディー
- 型番: GN-2139-2

Mini-mezzanine slot用の小型メザニンカードです。GN-2006-1にMIKUMARIポートを増設するために用います。GN-2006-4にはMIKUMARIポートがフロントパネルに存在しますが、CRVを取り付けて更にMIKUMARIポートを増やすことも可能です。
1000BASE-SXかLX用の光ファイバーモジュールを使用してください。LANケーブル用のモジュールは使用できません。
