---
title: データの冗長性
titleSuffix: Azure Storage
description: Azure Storage でのデータ冗長性について説明します。 Microsoft Azure Storage アカウント内のデータは、持続性と高可用性を保証するため、レプリケートされます。
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 08/18/2021
ms.author: tamram
ms.subservice: common
ms.openlocfilehash: f6de961bce052818270a949b45ed821d34b19b6c
ms.sourcegitcommit: 702df701fff4ec6cc39134aa607d023c766adec3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/03/2021
ms.locfileid: "131463299"
---
# <a name="azure-storage-redundancy"></a>Azure Storage の冗長性

Azure Storage では、計画されたイベントや計画外のイベント (一時的なハードウェア障害、ネットワークの停止または停電、大規模な自然災害など) から保護するため、常にデータの複数のコピーが格納されます。 冗長性を確保することで、障害が発生した場合でも、ストレージ アカウントが可用性と耐久性に関する目標を確実に達成できます。

ご自身のシナリオに最適な冗長性オプションを決定する際には、低コストと高可用性のトレードオフを検討してください。 どの冗長性オプションを選択するかの判断に役立つ要因は次のとおりです。

- プライマリ リージョンでのデータのレプリケート方法
- 地域災害から保護するため、プライマリ リージョンから地理的に離れている 2 番目のリージョンにデータをレプリケートするかどうか
- プライマリ リージョンが何らかの理由で使用できなくなった場合に、アプリケーションがセカンダリ リージョンのレプリケートされたデータへの読み取りアクセスを必要とするかどうか

> [!NOTE]
> この記事で説明する機能とリージョン別の提供状況は、階層型名前空間を持つアカウントでも利用できます。

## <a name="redundancy-in-the-primary-region"></a>プライマリ リージョンでの冗長性

Azure Storage アカウントのデータは、常にプライマリ リージョンで 3 回レプリケートされます。 Azure Storage には、プライマリ リージョンでデータをレプリケートする方法について、次の 2 つのオプションが用意されています。

- **ローカル冗長ストレージ (LRS)** は、プライマリ リージョンの 1 つの物理的な場所内で、データを同期的に 3 回コピーします。 LRS は最もコストのかからないレプリケーション オプションですが、高可用性または持続性を必要とするアプリケーションには推奨されません。
- **ゾーン冗長ストレージ (ZRS)** は、プライマリ リージョンの 3 つの Azure 可用性ゾーン間でデータを同期的にコピーします。 高可用性を必要とするアプリケーションでは、プライマリ リージョンで ZRS を使用し、セカンダリ リージョンにもレプリケートすることをお勧めします。

> [!NOTE]
> Microsoft では、Azure Data Lake Storage Gen2 ワークロード用にプライマリ リージョンで ZRS を使用することをお勧めします。

### <a name="locally-redundant-storage"></a>ローカル冗長ストレージ

ローカル冗長ストレージ (LRS) によって、プライマリ リージョンの単一のデータ センター内で、データは 3 回レプリケートされます。 LRS では、オブジェクトに年間 99.999999999% (9 が 11 個) 以上の持続性が提供されます。

LRS は、コストが最も安い冗長オプションであり、他のオプションと比較して持続性は最も低くなります。 LRS は、サーバー ラックとドライブの障害からデータを保護します。 ただし、データセンター内で火災や洪水などの災害が発生した場合、LRS を使用しているストレージ アカウントのすべてのレプリカが失われたり、回復不能になる可能性があります。 このリスクを軽減するために、Microsoft では[ゾーン冗長ストレージ](#zone-redundant-storage) (ZRS)、[geo 冗長ストレージ](#geo-redundant-storage) (GRS)、または [geo ゾーン冗長ストレージ](#geo-zone-redundant-storage) (GZRS) を使用することを推奨しています。

LRS を使用しているストレージ アカウントへの書き込み要求は、同期的に行われます。 書き込み操作は、データが 3 つのレプリカすべてに書き込まれた場合にのみ、正常に返されます。

次の図は、LRS を使用した、単一のデータ センター内でのデータのレプリケーション方法を示しています。

:::image type="content" source="media/storage-redundancy/locally-redundant-storage.png" alt-text="LRS を使用した、単一のデータ センター内でのデータのレプリケーション方法を示す図":::

LRS は、次のシナリオに適しています。

- データの損失が発生した場合に再構築が簡単なデータをアプリケーションで格納する場合は、LRS を選択することもできます。
- データ ガバナンスの要件によって、データのレプリケートが国内や地域内に制限されているアプリケーションの場合は、LRS を選択することができます。 場合によっては、データが geo レプリケーションされているペア リージョンが別の国または地域にある可能性があります。 ペアリングされるリージョンの詳細については、「[Azure リージョン](https://azure.microsoft.com/regions/)」を参照してください。

### <a name="zone-redundant-storage"></a>ゾーン冗長ストレージ

ゾーン冗長ストレージ (ZRS) は、プライマリ リージョンの 3 つの Azure 可用性ゾーン間で Azure Storage データを同期的にレプリケートします。 各可用性ゾーンは、独立した電源、冷却装置、ネットワークを備えた独立した物理的な場所です。 ZRS は、Azure Storage データ オブジェクトに年間 99.9999999999% (トゥエルブ ナイン) 以上の持続性を提供します。

ZRS を使用すると、データは、ゾーンが使用できなくなった場合でも読み取り操作と書き込み操作の両方にアクセスできます。 ゾーンが使用できなくなった場合、Azure により DNS の再指定などのネットワークの更新が行われます。 このような更新は、更新が完了する前にデータにアクセスすると、アプリケーションに影響を与える可能性があります。 ZRS 用のアプリケーションを設計するときは、指数バックオフを使用した再試行ポリシーを実装するなど、一時的な障害処理方法に関するプラクティスに従ってください。

ZRS を使用しているストレージ アカウントへの書き込み要求は、同期的に行われます。 書き込み操作は、3 つの可用性ゾーンのすべてのレプリカにデータが書き込まれた場合にのみ、正常に返されます。

高可用性を必要とするシナリオでは、プライマリ リージョンで ZRS を使用することをお勧めします。 データ ガバナンス要件を満たすために、国内または地域内へのデータのレプリケーションを制限する場合にも、ZRS をお勧めします。

次の図は、ZRS を使用した、プライマリ リージョンの可用性ゾーン間でのデータのレプリケーション方法を示しています。

:::image type="content" source="media/storage-redundancy/zone-redundant-storage.png" alt-text="ZRS を使用した、プライマリ リージョンでのデータのレプリケーション方法を示す図":::

ZRS は、データが一時的に使用できなくなった場合に、データに対して優れたパフォーマンス、低待機時間、および回復性を提供します。 ただし、ZRS 自体では、複数のゾーンが永続的に影響を受ける地域障害からデータを保護することはできません。 地域災害から保護するために、Microsoft では [geo ゾーン冗長ストレージ](#geo-zone-redundant-storage) (GZRS) を使用することを推奨しています。これはプライマリ リージョンで ZRS を使用し、セカンダリ リージョンにもデータを geo レプリケートします。

ZRS をサポートしているストレージ アカウントの種類とリージョンを、次の表に示します。

| ストレージ アカウントの種類 | サポートされているリージョン | サポートされているサービス |
|--|--|--|
| 汎用 v2<sup>1</sup> | (アフリカ) 南アフリカ北部<br /> (アジア太平洋) 東南アジア<br /> (アジア太平洋) オーストラリア東部<br /> (アジア太平洋) 東日本<br /> (カナダ) カナダ中部<br /> (ヨーロッパ) 北ヨーロッパ<br /> (ヨーロッパ) 西ヨーロッパ<br /> (ヨーロッパ) フランス中部<br /> (ヨーロッパ) ドイツ中西部<br /> (ヨーロッパ) 英国南部<br /> (南アメリカ) ブラジル南部<br /> (米国) 米国中部<br /> (米国) 米国東部<br /> (米国) 米国東部 2<br /> (米国) 米国中南部<br /> (米国) 米国西部 2 | ブロック blob<br /> ページ BLOB<sup>2</sup><br /> ファイル共有 (標準)<br /> テーブル<br /> キュー<br /> |
| Premium ブロック BLOB<sup>1</sup> | 東南アジア<br /> オーストラリア東部<br /> 北ヨーロッパ<br /> 西ヨーロッパ<br /> フランス中部 <br /> 東日本<br /> 英国南部 <br /> 米国東部 <br /> 米国東部 2 <br /> 米国西部 2| Premium ブロック BLOB のみ |
| Premium ファイル共有 | 東南アジア<br /> オーストラリア東部<br /> 北ヨーロッパ<br /> 西ヨーロッパ<br /> フランス中部 <br /> 東日本<br /> 英国南部 <br /> 米国東部 <br /> 米国東部 2 <br /> 米国西部 2 | Premium ファイル共有のみ |

<sup>1</sup> アーカイブ層は、ZRS アカウントでは現在サポートされていません。<br />
<sup>2</sup> 仮想マシン用の Azure マネージド ディスクを含むストレージ アカウントは、常に LRS を使用します。 Azure のアンマネージド ディスクでは、LRS も使用する必要があります。 GRS を使用する Azure アンマネージド ディスクのストレージ アカウントを作成することはできますが、非同期 geo レプリケーションの整合性に関する潜在的な問題のため、推奨されません。 マネージドとアンマネージド ディスクのどちらも ZRS または GZRS をサポートしていません。 マネージド ディスクの詳細については、[Azure マネージド ディスクの価格](https://azure.microsoft.com/pricing/details/managed-disks/)に関するページをご覧ください。

ZRS をサポートしているリージョンの詳細については、「[Azure の Availability Zones の概要](../../availability-zones/az-overview.md)」の「**リージョン別のサービスのサポート**」を参照してください。

## <a name="redundancy-in-a-secondary-region"></a>セカンダリ リージョンでの冗長性

高持続性を必要とするアプリケーションでは、プライマリ リージョンから数百キロ離れたセカンダリ リージョンにストレージ アカウントのデータを追加でコピーすることもできます。 ご使用のストレージ アカウントがセカンダリ リージョンにコピーされている場合は、地域全体が停電になった場合やプライマリ リージョンが復旧できない災害が発生しても、データは保持されます。

ストレージ アカウントの作成時に、アカウントのプライマリ リージョンを選択します。 ペアのセカンダリ リージョンはプライマリ リージョンに基づいて決定され、変更することはできません。 Azure でサポートされているリージョンの詳細については、「[Azure リージョン](https://azure.microsoft.com/global-infrastructure/regions/)」を参照してください。

Azure Storage には、セカンダリ リージョンにデータをコピーするための 2 つのオプションが用意されています。

- **geo 冗長ストレージ (GRS)** は、LRS を使用して、プライマリ リージョンの 1 つの物理的な場所内で、データを同期的に 3 回コピーします。 その後、セカンダリ リージョンの 1 つの物理的な場所にデータを非同期的にコピーします。 セカンダリ リージョン内のデータは、LRS を使用して同期的に 3 回コピーされます。
- **geo ゾーン冗長ストレージ (GZRS)** は、ZRS を使用して、プライマリ リージョンの 3 つの Azure 可用性ゾーン間でデータを同期的にコピーします。 その後、セカンダリ リージョンの 1 つの物理的な場所にデータを非同期的にコピーします。 セカンダリ リージョン内のデータは、LRS を使用して同期的に 3 回コピーされます。

> [!NOTE]
> GRS と GZRS の主な違いは、プライマリ リージョンでのデータのレプリケート方法です。 セカンダリ リージョンでは、データは常に LRS を使用して 3 回同期的にレプリケートされます。 セカンダリ リージョンの LRS は、ハードウェア障害からデータを保護します。

GRS または GZRS では、セカンダリ リージョンへのフェールオーバーがない限り、セカンダリ リージョンのデータを読み取りまたは書き込みアクセスに利用できません。 セカンダリ リージョンへの読み取りアクセスについては、読み取りアクセス geo 冗長ストレージ (RA-GRS) または読み取りアクセス geo ゾーン冗長ストレージ (RA-GZRS) を使用するようにストレージ アカウントを構成します。 詳細については、「[セカンダリ リージョンのデータへの読み取りアクセス](#read-access-to-data-in-the-secondary-region)」を参照してください。

プライマリ リージョンが使用できなくなった場合、セカンダリ リージョンへのフェールオーバーを選択できます。 フェールオーバーが完了すると、セカンダリ リージョンがプライマリ リージョンになり、データの読み取りと書き込みを再び行うことができます。 ディザスター リカバリーの詳細と、セカンダリ リージョンへのフェールオーバーの方法については、「[ディザスター リカバリーとストレージ アカウントのフェールオーバー](storage-disaster-recovery-guidance.md)」を参照してください。

> [!IMPORTANT]
> データはセカンダリ リージョンに非同期的にレプリケートされるため、プライマリ リージョンに影響を与える障害が発生すると、プライマリ リージョンを復旧できない場合にデータが失われる可能性があります。 プライマリ リージョンへの最新の書き込みと、セカンダリ リージョンへの最後の書き込みとの間隔は、回復ポイントの目標 (RPO) と呼ばれます。 RPO は、データを復旧できる対象の時点を示します。 現在、データをセカンダリ リージョンにレプリケートするのにかかる時間に関する SLA はありませんが、Azure Storage では通常、RPO は 15 分未満となります。

### <a name="geo-redundant-storage"></a>geo 冗長ストレージ

geo 冗長ストレージ (GRS) は、LRS を使用して、プライマリ リージョンの 1 つの物理的な場所内で、データを同期的に 3 回コピーします。 その後、プライマリ リージョンから何百キロも離れたセカンダリ リージョン内の 1 つの物理的な場所にデータが非同期にレプリケートされます。 GRS は、Azure Storage データ オブジェクトに年間 99.99999999999999% (シックスティーン ナイン) 以上の持続性を提供します。

書き込み操作は、まずプライマリの場所にコミットされ、LRS を使用してレプリケートされます。 更新は、セカンダリ リージョンに非同期にレプリケートされます。 データがセカンダリの場所に書き込まれると、LRS を使用してその場所内にレプリケートされます。

次の図は、GRS または RA-GRS を使用した、データのレプリケーション方法を示しています。

:::image type="content" source="media/storage-redundancy/geo-redundant-storage.png" alt-text="GRS または RA-GRS を使用した、データのレプリケーション方法を示す図":::

### <a name="geo-zone-redundant-storage"></a>geo ゾーン冗長ストレージ

geo ゾーン冗長ストレージ (GZRS) により、可用性ゾーン間での冗長性によって提供される高可用性と、geo レプリケーションによって提供されるリージョン障害からの保護が結合されます。 GZRS ストレージ アカウント内のデータは、プライマリ リージョンの 3 つの [Azure 可用性ゾーン](../../availability-zones/az-overview.md)間でコピーされ、地域災害から保護するためにセカンダリ リージョンにもレプリケートされます。 最大限の一貫性、持続性、高可用性、優れたパフォーマンス、ディザスター リカバリーのための回復性を必要とするアプリケーションに対しては、GZRS を使用することをお勧めします。

GZRS ストレージ アカウントを使用すると、可用性ゾーンが使用できなくなったり、回復できなくなった場合に、引き続きデータの読み取りと書き込みを行うことができます。 さらに、リージョン全体の障害が発生した場合や、プライマリ リージョンを回復できない災害が発生した場合にも、データは保持されます。 GZRS は、オブジェクトに年間 99.99999999999999% (シックスティーンナイン) 以上の持続性を確保するように設計されています。

次の図は、GZRS または RA-GZRS を使用した、データのレプリケーション方法を示しています。

:::image type="content" source="media/storage-redundancy/geo-zone-redundant-storage.png" alt-text="GZRS または RA-GZRS を使用した、データのレプリケーション方法を示す図":::

GZRS と RA-GZRS は、汎用 v2 ストレージ アカウントでのみサポートされます。 ストレージ アカウントの種類の詳細については、[Azure Storage アカウントの概要](storage-account-overview.md)に関するページを参照してください。 GZRS と RA-GZRS では、ブロック BLOB、ページ BLOB (VHD ディスクを除く)、ファイル、テーブル、およびキューがサポートされています。

GZRS と RA-GZRS は、次のリージョンでサポートされています。

- (アジア太平洋) 東アジア
- (アジア太平洋) 東南アジア
- (アジア太平洋) オーストラリア東部
- (アジア太平洋) 東日本
- (カナダ) カナダ中部
- (ヨーロッパ) 北ヨーロッパ
- (ヨーロッパ) 西ヨーロッパ
- (ヨーロッパ) フランス中部
- (ヨーロッパ) ノルウェー東部
- (ヨーロッパ) 英国南部
- (南アメリカ) ブラジル南部
- (米国) 米国中部
- (米国) 米国東部
- (米国) 米国東部 2
- (米国) 米国政府東部
- (米国) 米国中南部
- (米国) 米国西部 2
- (米国) 米国西部 3

価格については、[BLOB](https://azure.microsoft.com/pricing/details/storage/blobs)、[Files](https://azure.microsoft.com/pricing/details/storage/files/)、[Queues](https://azure.microsoft.com/pricing/details/storage/queues/)、[Tables](https://azure.microsoft.com/pricing/details/storage/tables/) の価格の詳細を参照してください。

## <a name="read-access-to-data-in-the-secondary-region"></a>セカンダリ リージョンのデータへの読み取りアクセス

geo 冗長ストレージ (GRS または GZRS を使用) は、リージョン障害から保護するために、セカンダリ リージョンの別の物理的な場所にデータをレプリケートします。 ただし、そのデータが読み取れるのは、お客様または Microsoft がプライマリ リージョンからセカンダリ リージョンへのフェールオーバーを開始した場合だけです。 セカンダリ リージョンへの読み取りアクセスを有効にすると、プライマリ リージョンが使用できなくなる状況を含め、読み取りにおいて常にデータを使用することができます。 セカンダリ リージョンへの読み取りアクセスについては、読み取りアクセス geo 冗長ストレージ (RA-GRS) または読み取りアクセス geo ゾーン冗長ストレージ (RA-GZRS) を有効にします。

> [!NOTE]
> Azure Files では、読み取りアクセス geo 冗長ストレージ (RA-GRS) と読み取りアクセス geo ゾーン冗長ストレージ (RA-GZRS) はサポートされていません。

### <a name="design-your-applications-for-read-access-to-the-secondary"></a>セカンダリへの読み取りアクセス用にアプリケーションを設計する

ご使用のストレージ アカウントがセカンダリ リージョンへの読み取りアクセス用に構成されている場合、プライマリ リージョンが何らかの理由で使用できなくなった場合に、セカンダリ リージョンからデータを読み取るようにシームレスに変更するようにアプリケーションを設計できます。

RA-GRS または RA-GZRS を有効にした後はセカンダリ リージョンを読み取りアクセスのために使用できるため、アプリケーションを事前にテストして、停止が発生した場合にセカンダリから正しく読み取りが行われることを確認できます。 geo 冗長性を利用するようにアプリケーションを設計する方法の詳細については、「[geo 冗長性を使用して高可用性アプリケーションを設計する](geo-redundant-design.md)」を参照してください。

セカンダリへの読み取りアクセスが有効になっていると、アプリケーションはプライマリ エンドポイントからだけでなく、セカンダリ エンドポイントからも読み取れます。 セカンダリ エンドポイントでは、アカウント名にサフィックス " *–secondary*" が追加されます。 たとえば、BLOB ストレージのプライマリ エンドポイントが `myaccount.blob.core.windows.net` の場合、セカンダリ エンドポイントは `myaccount-secondary.blob.core.windows.net` になります。 ストレージ アカウントのアカウント アクセス キーは、プライマリ エンドポイントとセカンダリ エンドポイントの両方で同じです。

### <a name="check-the-last-sync-time-property"></a>最終同期時刻プロパティを確認する

データはセカンダリ リージョンに非同期的にレプリケートされるため、セカンダリ リージョンは多くの場合、プライマリ リージョンより遅れています。 プライマリ リージョンで障害が発生した場合、プライマリへのすべての書き込みがまだセカンダリにレプリケートされていない可能性があります。

セカンダリ リージョンにレプリケートされた書き込み操作を特定するために、アプリケーションでストレージ アカウントの **[最終同期時刻]** プロパティを確認することができます。 最終同期時刻より前にプライマリ リージョンに書き込まれたすべての書き込み操作は、セカンダリ リージョンに正常にレプリケートされています。つまり、セカンダリ リージョンからの読み取りが可能です。 最終同期時刻の後にプライマリ リージョンに書き込まれた書き込み操作は、セカンダリ リージョンにレプリケートされている場合とそうでない場合があります。つまり、読み取り操作で使用できない場合があります。

**[最終同期時刻]** プロパティの値は、Azure PowerShell、Azure CLI、または Azure Storage クライアント ライブラリのいずれかを使用してクエリできます。 **[最終同期時刻]** プロパティは、GMT 日付/時刻の値です。 詳細については、「[ストレージ アカウントの最終同期時刻プロパティを確認する](last-sync-time-get.md)」を参照してください。

## <a name="summary-of-redundancy-options"></a>冗長オプションの概要

次のセクションの表は、Azure Storage で使用可能な冗長オプションをまとめています。

### <a name="durability-and-availability-parameters"></a>耐久性と可用性パラメーター

次の表では、各冗長オプションの主要なパラメーターについて説明します。

| パラメーター | LRS | ZRS | GRS/RA-GRS | GZRS/RA-GZRS |
|:-|:-|:-|:-|:-|
| 指定された 1 年間にわたるオブジェクトの持続性のパーセンテージ | 99.999999999% (イレブン ナイン) 以上 | 99.9999999999% (トゥエルブ ナイン) 以上 | 99.99999999999999% (シックスティーン ナイン) 以上 | 99.99999999999999% (シックスティーン ナイン) 以上 |
| 読み取り要求の可用性 | 99.9% 以上 (クール アクセス層の場合、99%) | 99.9% 以上 (クール アクセス層の場合、99%) | GRS の場合、99.9% 以上 (クール アクセス層の場合、99%)<br /><br />RA-GRS の場合、99.99% 以上 (クール アクセス層の場合、99.9%) | GZRS の場合、99.9% 以上 (クール アクセス層の場合、99%)<br /><br />RA-GZRS の場合、99.99% 以上 (クール アクセス層の場合、99.9%) |
| 書き込み要求の可用性 | 99.9% 以上 (クール アクセス層の場合、99%) | 99.9% 以上 (クール アクセス層の場合、99%) | 99.9% 以上 (クール アクセス層の場合、99%) | 99.9% 以上 (クール アクセス層の場合、99%) |
| 個別のノードで保持されるデータ コピーの数 | 1 つのリージョン内に 3 つのコピー | 1 つのリージョン内の個別の可用性ゾーン間で 3 つのコピー | プライマリ リージョンに 3 つ、セカンダリ リージョンに 3 つ、合計 6 つのコピー | プライマリ リージョンの個別の可用性ゾーン間で 3 つ、セカンダリ リージョンに 3 つのローカル冗長コピー、合計 6 つのコピー |

### <a name="durability-and-availability-by-outage-scenario"></a>障害のシナリオでの耐久性と可用性

次の表は、特定のシナリオにおいてデータに耐久性と可用性があるかどうかを、ご使用のストレージ アカウントに対して有効な冗長性の種類に応じて示しています。

| 障害のシナリオ | LRS | ZRS | GRS/RA-GRS | GZRS/RA-GZRS |
|:-|:-|:-|:-|:-|
| データ センター内のノードが使用できなくなる | はい | はい | はい | はい |
| データ センター全体 (ゾーンまたは非ゾーン) が使用できなくなる | いいえ | はい | 可<sup>1</sup> | はい |
| プライマリ リージョンでリージョン全体の障害が発生する | いいえ | いいえ | 可<sup>1</sup> | 可<sup>1</sup> |
| プライマリ リージョンが使用できなくなった場合、セカンダリ リージョンに対する読み取りアクセスが可能 | いいえ | いいえ | はい (RA-GRS を使用) | はい (RA-GZRS を使用) |

<sup>1</sup>プライマリ リージョンが使用できなくなった場合に書き込みの可用性を復元するには、アカウントのフェールオーバーが必要です。 詳細については、「[ディザスター リカバリーとストレージ アカウントのフェールオーバー](storage-disaster-recovery-guidance.md)」を参照してください。

### <a name="supported-azure-storage-services"></a>サポートされている Azure Storage サービス

次の表は、Azure Storage サービスごとにサポートされる冗長性オプションを示しています。

| LRS | ZRS | GRS | RA-GRS | GZRS | RA-GZRS |
|---|---|---|---|---|---|
| BLOB ストレージ <br />ストレージ <br />テーブル ストレージ <br />Azure Files<sup>1、</sup><sup>2</sup> <br />Azure Managed Disks | BLOB ストレージ <br />ストレージ <br />テーブル ストレージ <br />Azure Files<sup>1、</sup><sup>2</sup> | BLOB ストレージ <br />ストレージ <br />テーブル ストレージ <br />Azure Files<sup>1</sup> | BLOB ストレージ <br />ストレージ <br />テーブル ストレージ <br /> | BLOB ストレージ <br />ストレージ <br />テーブル ストレージ <br />Azure Files<sup>1</sup> | BLOB ストレージ <br />ストレージ <br />テーブル ストレージ <br /> |

<sup>1</sup> 標準ファイル共有は LRS と ZRS でサポートされます。 標準ファイル共有は、サイズが 5 TiB 以下である場合に限り、GRS と GZRS でサポートされます。<br />
<sup>2</sup> Premium ファイル共有は LRS と ZRS でサポートされます。<br />

### <a name="supported-storage-account-types"></a>サポートされるストレージ アカウントの種類

次の表は、ストレージ アカウントの種類ごとにサポートされる冗長性オプションを示しています。 ストレージ アカウントの種類については、「[ストレージ アカウントの概要](storage-account-overview.md)」を参照してください。

| LRS | ZRS | GRS/RA-GRS | GZRS/RA-GZRS |
|:-|:-|:-|:-|
| 汎用 v2<br /> 汎用 v1<br /> Premium ブロック BLOB<br /> レガシ BLOB<br /> Premium ファイル共有 | 汎用 v2<br /> Premium ブロック BLOB<br /> Premium ファイル共有 | 汎用 v2<br /> 汎用 v1<br /> レガシ BLOB | 汎用 v2 |

ストレージ アカウントの冗長オプションに従って、すべてのストレージ アカウントのすべてのデータがコピーされます。 ブロック BLOB、追加 BLOB、ページ BLOB、キュー、テーブル、ファイルなどのオブジェクトがコピーされます。 すべての層 (アーカイブ層を含む) のデータがコピーされます。 BLOB 層の詳細については、[BLOB データのホット、クール、アーカイブ アクセス層](../blobs/access-tiers-overview.md)に関するページを参照してください。

各冗長オプションの料金情報については、[Azure Storage の価格](https://azure.microsoft.com/pricing/details/storage/)に関するページをご覧ください。

> [!NOTE]
> 現在、Azure Premium Disk Storage は、ローカル冗長ストレージ (LRS) のみをサポートしています。 ブロック BLOB ストレージ アカウントでは、特定のリージョンで、ローカル冗長ストレージ (LRS) とゾーン冗長ストレージ (ZRS) をサポートしています。

## <a name="data-integrity"></a>データ整合性

Azure Storage では、巡回冗長検査 (CRCs) を使用して、格納データの整合性を定期的に検証します。 データの破損が検出された場合は、冗長データを使用して修復されます。 また Azure Storage では、データの格納時または取得時にデータ パケットの破損を検出する目的ですべてのネットワーク トラフィックのチェックサムを計算します。

## <a name="see-also"></a>関連項目

- [ストレージ アカウントの最終同期時刻プロパティを確認する](last-sync-time-get.md)
- [ストレージ アカウントの冗長オプションを変更する](redundancy-migration.md)
- [geo 冗長性を使用する高可用性アプリケーションの設計](geo-redundant-design.md)
- [ディザスター リカバリーとストレージ アカウントのフェールオーバー](storage-disaster-recovery-guidance.md)
