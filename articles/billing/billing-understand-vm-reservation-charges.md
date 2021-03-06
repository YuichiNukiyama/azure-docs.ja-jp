---
title: "Azure 予約仮想マシン インスタンスの割引の適用について | Microsoft Docs"
description: "Azure 予約 VM インスタンスの割引が、実行中の VM にどのように適用されるのかを説明します。"
services: billing
documentationcenter: 
author: vikramdesai01
manager: vikdesai
editor: 
ms.service: billing
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/10/2017
ms.author: vikdesai
ms.openlocfilehash: 3fd12bd3c51eeef57c896da030a83e447dc3e8ff
ms.sourcegitcommit: 7d107bb9768b7f32ec5d93ae6ede40899cbaa894
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/16/2017
---
# <a name="understand-how-the-reserved-virtual-machine-instance-discount-is-applied"></a>予約仮想マシン インスタンスの割引の適用方法について
予約 VM インスタンスを購入すると、予約の属性や数量に合致する仮想マシンに対して予約の割引が自動的に適用されます。 予約購入分は、ご利用の仮想マシンのインフラストラクチャ コストに充当されます。 以下の表は、予約を購入した後の仮想マシンのコストを示しています。 ストレージとネットワークについては、すべての場合において通常料金が適用されます。

| 仮想マシンのタイプ  | 予約後の料金 |    
|-----------------------|--------------------------------------------| 
|Linux VM (追加ソフトウェアなし) | 予約購入分は、ご利用の VM インフラストラクチャ コストに充当されます。|
|Linux VM + ソフトウェア料金 (例: Red Hat) | 予約購入分は、インフラストラクチャ コストに充当されます。 追加ソフトウェアについては課金の対象となります。|
|Windows VM (追加ソフトウェアなし) |予約購入分は、インフラストラクチャ コストに充当されます。 Windows ソフトウェアについては課金の対象となります。|
|Windows VM + 追加ソフトウェア (例: SQL Server) | 予約購入分は、インフラストラクチャ コストに充当されます。 Windows ソフトウェアや追加ソフトウェアについては課金の対象となります。|
|Windows VM + [Azure ハイブリッド特典](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing) | 予約購入分は、インフラストラクチャ コストに充当されます。 Windows ソフトウェアの料金には、Azure ハイブリッド特典が適用されます。 その他のソフトウェアについては、別途課金の対象となります。| 

## <a name="application-of-reservation-discount-to-non-windows-vms"></a>非 Windows VM への予約割引の適用
 予約割引は、実行中の VM インスタンスに 1 時間単位で適用されます。 購入済みの予約は、実行中の VM によって生成された使用量と照合され、予約割引が適用されます。 以下のグラフは、課金対象の VM 使用量に予約を適用した例を示しています。 この例は、単一の予約購入で、合致する VM インスタンスが 2 つあることを前提としています。

![予約 VM インスタンスの適用](media/billing-reserved-vm-instance-application/billing-reserved-vm-instance-application.png)

1.  予約 VM インスタンスのラインを超える使用量には、通常の従量課金料金が適用されます。 予約 VM インスタンスのラインを下回る使用量については、予約購入分として前払い済みであるため課金されません。
2.  Hour 1 では、インスタンス 1 の実行時間が 0.75 時間、インスタンス 2 の実行時間が 0.5 時間です。 Hour 1 の合計使用量は 1.25 時間となります。 残りの 0.25 時間については従量課金料金が発生します。
3.  Hour 2 と Hour 3 では、どちらのインスタンスも実行時間はそれぞれ 1 時間です。 一方のインスタンスには予約購入分が充当されますが、もう一方には従量課金料金が発生します。
4.  Hour 4 では、インスタンス 1 の実行時間が 0.5 時間、インスタンス 2 の実行時間が 1 時間です。 インスタンス 1 は予約購入分で全額充当されます。またインスタンス 2 の 0.5 時間も充当されます。 残りの 0.5 時間については従量課金料金が発生します。

予約の適用を課金の使用状況レポートの観点から理解し、把握するには、[予約 VM インスタンスの使用量の概要](https://go.microsoft.com/fwlink/?linkid=862757)に関するページを参照してください。

## <a name="application-of-reservation-discount-to-windows-vms"></a>Windows VM への予約割引の適用
Windows VM インスタンスの実行中は、インフラストラクチャ コストに予約が適用されて充当されます。 VM インフラストラクチャ コストに対する予約の適用は、Windows VM の場合も 非 Windows VM の場合も変わりません。 Windows ソフトウェアには別途、vCPU 単位の料金が発生します。 [予約に伴う Windows ソフトウェアのコスト](https://go.microsoft.com/fwlink/?linkid=862756)に関するページを参照してください。 Windows ライセンスの費用は、[Windows Server 向け Azure ハイブリッド特典](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/hybrid-use-benefit-licensing)で充当することができます。

## <a name="need-help-contact-support"></a>お困りの際は、 サポートにお問い合せください

お困りの際は、問題を迅速に解決するために、[サポートにお問い合わせ](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)ください。
