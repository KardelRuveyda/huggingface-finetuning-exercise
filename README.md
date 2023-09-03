{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "provenance": [],
      "authorship_tag": "ABX9TyOpUN3H88xHgZFkyfcvKHsx",
      "include_colab_link": true
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "language_info": {
      "name": "python"
    }
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "view-in-github",
        "colab_type": "text"
      },
      "source": [
        "<a href=\"https://colab.research.google.com/gist/KardelRuveyda/a9c0e02f3a862fc5b7ef8e6281e43068/huggingface-finetuning-exercise.ipynb\" target=\"_parent\"><img src=\"https://colab.research.google.com/assets/colab-badge.svg\" alt=\"Open In Colab\"/></a>"
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "import pandas as pd\n",
        "from transformers import GPT2Tokenizer, GPT2LMHeadModel\n",
        "from transformers import Trainer, TrainingArguments\n",
        "from transformers import GPT2Tokenizer, GPT2LMHeadModel\n",
        "from transformers import AutoTokenizer, AutoModelForCausalLM\n",
        "from transformers import TextDataset, DataCollatorForLanguageModeling, Trainer, TrainingArguments\n",
        "from datasets import load_dataset\n",
        "import torch\n",
        "\n",
        "\n",
        "\n",
        "\n",
        "# Excel dosyasını oku\n",
        "df = pd.read_excel(\"train-report.xlsm\")  # Excel dosyanızın adını burada belirtin\n",
        "\n",
        "# Verileri bir metin dosyasına kaydedin\n",
        "df.to_csv('veri_kumesi.txt', index=False, header=False, sep='\\t', encoding='utf-8')\n"
      ],
      "metadata": {
        "id": "PyrxSD8SWMTn"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# \"cümle\" ve \"intentadi\" sütunlarını seçin\n",
        "cumleler = df[\"Cumle\"].astype(str).tolist()\n",
        "intentler = df[\"IntentAdi\"].astype(str).tolist()\n",
        "\n",
        "\n",
        "print(cumleler);\n",
        "print(intentler);"
      ],
      "metadata": {
        "id": "Vv4_QEa5WSF3",
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "outputId": "15584110-2383-4b75-9b6f-2e444ace550a"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "['53293112028', 'ruveydakardelcetin@gmail.com', 'maraba', 'kardel.cetin@d-teknoloji.com.tr', 'aksam', 'aafasfwef', '15.02.2000', 'dsgadsgf', 'aasdada', 'aaa', 'asd', 'ccc', 'bodrum canli', 'anlayamadım', 'ne demek istediğinizi anlamadım', 'ana menüye dön', 'ana menü', 'merhaba', 'mrb', 'naber', 'sa', 'merhabalar', 'selamlar', 'merhaba', 'Selam', 'selamün aleyküm', 'merhaba canım', 'Merhaba', 'Selamın aleyküm', 'selamın aleyküm', 'ne demek istiyorsunuz?', 'bay bay', 'görüşürüz', 'görüşmek üzere', 'by', 'bye', 'görüşrüüz', 'güle güle', 'bayy', 'iyi akşamlar', 'iyi günler', 'eyvallah', 'YNT SİGORTA ACENTELİĞİ LTD. ŞTİ.', 'YAY10', 'YAY6', 'Yatarak', 'satın alma yapmak isityorum', 'satın al', 'satış gerçekleştirmek istiyorum', 'sigorta yaptırmadım', 'sigorta yaptırmak istiyorum', 'teklifi satın almak istiyorum', 'sigortam mevcut değil adres bilgilerini girmek istiyorum', 'sigortam yok', 'sigortalı değilim', 'teklifi satışa dönüştürmek istiyorum', '5305153061', 'evet', 'evettt', 'evt', 'olur', 'onaylıyorum', 'olsun', 'okey', 'tamamdır', 'tmm', 'yes', 'iyi olur', 'ok', 'okey', 'tamam', 'hayir', 'hayır', 'hayırrr', 'hyr', 'istemiyorum', 'no', 'noo', 'onaylamıyorum', 'nayır', 'olmaz', 'yok olmaz', 'yok', 'hayır', 'no', 'olmaz', 'yok', 'teklifi kabul ediyorum satışa geçmek istiyorum', '<intent_modeli_cumleleri>', 'hayırr', 'hayirr', 'bulunmuyor', 'bulunmuyo', 'bulunmyr', 'bulunuyor', 'bulunuyo', 'by', 'bye', 'görüşürüz', 'güle güle', 'hoşçakal', 'sonlandır', 'neyse kalsın', 'bye bye', 'iptal', 'iptalet', 'kes', 'kes tamam', 'by by', 'bye', 'görüşmek dileğiyle', 'gorusmek uzere', 'görüşmek üzere', 'güle güle', 'hadi by', 'hoşçakalın', 'iyi akşamlar', 'iyi çalışmalar', 'iyi günler dilerim', 'kendinize iyi bakın', 'kib', 'elveda', 'görüşürüz', 'hoşçakal', 'hoşça kal', 'baybay', 'görüşmek üzere', '173', 'selam', 'görüşmek üzere', 'hoşça kal', 'ekbilgiler', 'Teklif', 'Yeni Teklif', 'TSS', 'TSS Teklif', 'Tamamlayıcı Sağlık Sigortası Teklif', 'iptal', 'iptalll', 'iptal et', 'iptal ya', 'ailem için teklif almak istiyorum odağım yay6 olsun', 'ailem için teklif almak istiyorum odağım yay10 olsun', 'yay10 teklif almak isterim', 'ailem için teklif almak isteirm', 'ailem için teklif almak istiyorum odağım yatarak olsun', 'yardım almak istiyorum', 'kaydet lütfen', 'Gönder', 'gönder', 'gönderir misin', 'gönderrr', 'gönder gelsin', 'kaydet', 'kaydettt', 'hello', 'hay', 'hy', 'hallo', '53293112028', '5305153061', 'kardel@gmail.com', 'test@gmail.com', '1995-01-07 00:00:00', 'tklf', 'teklifff', 'teklif almak isterim', 'teklif alabilir imyim', 'teklif almak istiyorum', 'teklif versene', 'taklif', 'tekliff', 'teklifffff alayıımm', 'bana bi teklif', 'haydi bi teklif ver bakalım', '54', '52', '64', 'asjhdşahj', '763asdjash', 'asdg76232', 'jhasdsad764', '93aosud', '21', '3432', 'asdjhaşdh093', 'aldhs56', 'kardeşim için teklif almak istiyorum', 'güney amerika seyahatı için sigorta yaptıracaktım', '78', '8721837', 'oasdoad8989']\n",
            "['diger', 'diger', 'selamlama', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'diger', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'diger', 'diger', 'diger', 'diger', 'satinal', 'satinal', 'satinal', 'ekbilgiler', 'ekbilgiler', 'satinal', 'ekbilgiler', 'ekbilgiler', 'ekbilgiler', 'satinal', 'diger', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Onaylama', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'satinal', 'SorunBildirimi', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Reddetme', 'Onaylama', 'Onaylama', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'diger', 'selamlama', 'vedalasma', 'vedalasma', 'ekbilgiler', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'vedalasma', 'vedalasma', 'vedalasma', 'vedalasma', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'tekliftekrarkaydet', 'onaykodutekrar', 'onaykodutekrar', 'onaykodutekrar', 'onaykodutekrar', 'onaykodutekrar', 'tekliftekrarkaydet', 'tekliftekrarkaydet', 'selamlama', 'selamlama', 'selamlama', 'selamlama', 'diger', 'diger', 'diger', 'diger', 'diger', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'yeniteklifgiris', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'diger', 'yeniteklifgiris', 'yeniteklifgiris', 'diger', 'diger', 'diger']\n"
          ]
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "#redrussianarmy/gpt2-turkish-cased\n",
        "model_name = \"redrussianarmy/gpt2-turkish-cased\"\n",
        "tokenizer = AutoTokenizer.from_pretrained(model_name)\n",
        "model = AutoModelForCausalLM.from_pretrained(model_name)\n"
      ],
      "metadata": {
        "id": "7yVkcM31WTAC",
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "outputId": "9c274f15-caf9-48a8-e3ad-7cea0db3e77e"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stderr",
          "text": [
            "Special tokens have been added in the vocabulary, make sure the associated word embeddings are fine-tuned or trained.\n"
          ]
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "tokenizer.pad_token = tokenizer.eos_token\n",
        "\n",
        "# Verileri tokenlara çevirme\n",
        "encoding = tokenizer(cumleler, return_tensors=\"pt\", padding=True, truncation=True)"
      ],
      "metadata": {
        "id": "LgnP1haoWWjC"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# Veri kümesini oluşturma\n",
        "dataset = TextDataset(\n",
        "    tokenizer=tokenizer,\n",
        "    file_path=\"veri_kumesi.txt\",  # Veri kümesi dosyasının yolunu belirtin\n",
        "    block_size=128,  # Metin bloğunun boyutunu belirleyin\n",
        ")\n",
        "\n",
        "# Veri kümesindeki örnek sayısını alın\n",
        "num_examples = len(dataset)\n",
        "\n",
        "print(num_examples)\n",
        "\n",
        "# İlk 10 örneği görüntüle\n",
        "for i in range(min(10, num_examples)):  # Eğer veri kümesi 10'dan az örnek içeriyorsa, tüm örnekleri görüntüleyin\n",
        "    print(dataset[i])\n",
        "\n"
      ],
      "metadata": {
        "id": "BtkQNGl0Wjfl",
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "outputId": "f7d3c2e4-dbc3-483a-a572-3f9632ca3b63"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "16\n",
            "tensor([ 4310,  1959,  3132, 10232,  2078,   197,    67,  8254,   198,   622,\n",
            "         3304,    67,   461,   446,   417,    66,   316,   259,    31, 14816,\n",
            "           13,   785,   197,    67,  8254,   198,  3876, 15498,   197,   741,\n",
            "        43695,  1689,   198,    74,   446,   417,    13,    66,   316,   259,\n",
            "           31,    67,    12,   660, 15418, 14057,  7285,    13,   785,    13,\n",
            "         2213,   197,    67,  8254,   198,  4730,   321,   197,    67,  8254,\n",
            "          198,    64,  1878,   292,    69,   732,    69,   197,    67,  8254,\n",
            "          198,  1314,    13,  2999,    13, 11024,   197,    67,  8254,   198,\n",
            "         9310,    70,  5643,    70,    69,   197,    67,  8254,   198,    64,\n",
            "          292,    67,  4763,   197,    67,  8254,   198, 46071,   197,    67,\n",
            "         8254,   198,   292,    67,   197,    67,  8254,   198,   535,    66,\n",
            "          197,    67,  8254,   198,    65,   375,  6582,   460,  4528,   197,\n",
            "           67,  8254,   198,   272, 10724,   321,   324, 30102])\n",
            "tensor([   76,   197,    67,  8254,   198,   710,  1357,   988,   318,  1513,\n",
            "           72, 33133,   259,   528,    72,   281,  2543,   324, 30102,    76,\n",
            "          197,    67,  8254,   198,  2271,  1450,  9116,  5948,   288, 48863,\n",
            "          197,   741, 43695,  1689,   198,  2271,  1450,  9116,   197,   741,\n",
            "        43695,  1689,   198,   647,  5976,    64,   197,   741, 43695,  1689,\n",
            "          198,    76, 26145,   197,   741, 43695,  1689,   198,    77, 27359,\n",
            "          197,   741, 43695,  1689,   198, 11400,   197,   741, 43695,  1689,\n",
            "          198,   647,  5976,   282,   283,   197,   741, 43695,  1689,   198,\n",
            "          741,   321, 21681,   197,   741, 43695,  1689,   198,   647,  5976,\n",
            "           64,   197,   741, 43695,  1689,   198, 48767,   321,   197,   741,\n",
            "        43695,  1689,   198,   741,   321,  9116,    77,   257,  1636,    74,\n",
            "         9116,    76,   197,   741, 43695,  1689,   198,   647,  5976,    64,\n",
            "          460, 30102,    76,   197,   741, 43695,  1689,   198])\n",
            "tensor([13102,  5976,    64,   197,   741, 43695,  1689,   198, 48767,   321,\n",
            "        30102,    77,   257,  1636,    74,  9116,    76,   197,   741, 43695,\n",
            "         1689,   198,   741,   321, 30102,    77,   257,  1636,    74,  9116,\n",
            "           76,   197,   741, 43695,  1689,   198,   710,  1357,   988,   318,\n",
            "           83,  7745,   669,   403, 10277,    30,   197,    67,  8254,   198,\n",
            "        24406, 15489,   197,  1079,   282, 11797,   198,    70, 30570,  9116,\n",
            "        46481, 25151,  9116,    89,   197,  1079,   282, 11797,   198,    70,\n",
            "        30570,  9116, 46481,    76,   988,  6184,   120,    89,   567,   197,\n",
            "         1079,   282, 11797,   198,  1525,   197,  1079,   282, 11797,   198,\n",
            "        16390,   197,  1079,   282, 11797,   198,    70, 30570,  9116, 46481,\n",
            "           81,  9116,  9116,    89,   197,  1079,   282, 11797,   198,    70,\n",
            "         9116,   293,   308,  9116,   293,   197,  1079,   282, 11797,   198,\n",
            "        24406,    88,   197,  1079,   282, 11797,   198,  7745])\n",
            "tensor([   72, 47594, 46481,   321, 21681,   197,  1079,   282, 11797,   198,\n",
            "         7745,    72,   308,  9116,    77,  1754,   197,  1079,   282, 11797,\n",
            "          198,  2959,    85, 31840,   197,  1079,   282, 11797,   198,    56,\n",
            "        11251,   311,   128,   108,    38,  1581,  5603,  7125,  3525,  3698,\n",
            "          128,   108,   128,   252,   128,   108, 42513,    13, 25370,   252,\n",
            "           51,   128,   108,    13,   197,    67,  8254,   198,    56,  4792,\n",
            "          940,   197,    67,  8254,   198,    56,  4792,    21,   197,    67,\n",
            "         8254,   198,    56,  9459,   461,   197,    67,  8254,   198, 49720,\n",
            "        30102,    77,   435,  2611,   331,   499,    76,   461,   318,   414,\n",
            "        19220,   197, 49720,  1292,   198, 49720, 30102,    77,   435,   197,\n",
            "        49720,  1292,   198, 49720, 30102, 46481, 27602, 16175,   988,   293,\n",
            "        46481,    83,  2533,   988,   318,    83,  7745, 19220,   197, 49720,\n",
            "         1292,   198,    82,   328,   419,    64,   331,  2373])\n",
            "tensor([30102,    81,  9937, 30102,    76,   197,   988, 33473,    70,  5329,\n",
            "          198,    82,   328,   419,    64,   331,  2373, 30102, 26224,   461,\n",
            "          318,    83,  7745, 19220,   197,   988, 33473,    70,  5329,   198,\n",
            "        35424,    75, 22238,  3332, 30102,    77,   435,    76,   461,   318,\n",
            "           83,  7745, 19220,   197, 49720,  1292,   198,    82,   328,   419,\n",
            "          321,   502,    85,  8968,   390, 33133,   346,   512,   411, 47027,\n",
            "           70,  5329,  5362,   308,  2533,   988,   318,    83,  7745, 19220,\n",
            "          197,   988, 33473,    70,  5329,   198,    82,   328,   419,   321,\n",
            "          331,   482,   197,   988, 33473,    70,  5329,   198,    82,   328,\n",
            "        16906, 30102,   390, 33133,   346,   320,   197,   988, 33473,    70,\n",
            "         5329,   198, 35424,    75, 22238,  3332, 30102, 46481,    64,   288,\n",
            "        48863,  9116, 46481,    83, 25151,    76,   988,   318,    83,  7745,\n",
            "        19220,   197, 49720,  1292,   198,    20, 22515,  1314])\n",
            "tensor([ 1270,  5333,   197,    67,  8254,   198,    68, 16809,   197,  2202,\n",
            "          323,    75,  1689,   198, 44655,   926,    83,   197,  2202,   323,\n",
            "           75,  1689,   198,  1990,    83,   197,  2202,   323,    75,  1689,\n",
            "          198,   349,   333,   197,  2202,   323,    75,  1689,   198,   261,\n",
            "          323,    75, 30102,    88, 19220,   197,  2202,   323,    75,  1689,\n",
            "          198, 10220,   403,   197,  2202,   323,    75,  1689,   198,  2088,\n",
            "           88,   197,  2202,   323,    75,  1689,   198,    83,   321, 28745,\n",
            "        30102,    81,   197,  2202,   323,    75,  1689,   198,    83,  3020,\n",
            "          197,  2202,   323,    75,  1689,   198,  8505,   197,  2202,   323,\n",
            "           75,  1689,   198,  7745,    72, 25776,   333,   197,  2202,   323,\n",
            "           75,  1689,   198,   482,   197,  2202,   323,    75,  1689,   198,\n",
            "         2088,    88,   197,  2202,   323,    75,  1689,   198,    83,   321,\n",
            "          321,   197,  2202,   323,    75,  1689,   198,    71])\n",
            "tensor([  323,   343,   197, 32259,   316,  1326,   198,    71,   323, 30102,\n",
            "           81,   197, 32259,   316,  1326,   198,    71,   323, 30102, 21062,\n",
            "           81,   197, 32259,   316,  1326,   198,    71,  2417,   197, 32259,\n",
            "          316,  1326,   198,   396,   368,  7745, 19220,   197, 32259,   316,\n",
            "         1326,   198,  3919,   197, 32259,   316,  1326,   198,    77,  2238,\n",
            "          197, 32259,   316,  1326,   198,   261,   323,  2543, 30102,    88,\n",
            "        19220,   197, 32259,   316,  1326,   198,    77,   323, 30102,    81,\n",
            "          197, 32259,   316,  1326,   198,   349,    76,  1031,   197, 32259,\n",
            "          316,  1326,   198,    88,   482, 25776,    76,  1031,   197, 32259,\n",
            "          316,  1326,   198,    88,   482,   197, 32259,   316,  1326,   198,\n",
            "           71,   323, 30102,    81,   197, 32259,   316,  1326,   198,  3919,\n",
            "          197, 32259,   316,  1326,   198,   349,    76,  1031,   197, 32259,\n",
            "          316,  1326,   198,    88,   482,   197, 32259,   316])\n",
            "tensor([ 1326,   198, 35424,    75, 22238,   479, 16665,  1225,  7745, 19220,\n",
            "         3332, 30102, 46481,    64,  4903, 16175,    76,   988,   318,    83,\n",
            "         7745, 19220,   197, 49720,  1292,   198,    27, 48536,    62, 19849,\n",
            "           72,    62, 36340,   293,  1754,    72,    29,   197,    50,   273,\n",
            "          403,    33,   688,   343, 25236,   198,    71,   323, 30102, 21062,\n",
            "          197, 32259,   316,  1326,   198,    71,   323,   343,    81,   197,\n",
            "        32259,   316,  1326,   198, 15065,   403,    76,  4669,   273,   197,\n",
            "        32259,   316,  1326,   198, 15065,   403,    76,  4669,    78,   197,\n",
            "        32259,   316,  1326,   198, 15065,   403,  1820,    81,   197, 32259,\n",
            "          316,  1326,   198, 15065,   403,  4669,   273,   197,  2202,   323,\n",
            "           75,  1689,   198, 15065,   403,  4669,    78,   197,  2202,   323,\n",
            "           75,  1689,   198,  1525,   197,  1079,   282, 11797,   198, 16390,\n",
            "          197,  1079,   282, 11797,   198,    70, 30570,  9116])\n",
            "tensor([46481, 25151,  9116,    89,   197,  1079,   282, 11797,   198,    70,\n",
            "         9116,   293,   308,  9116,   293,   197,  1079,   282, 11797,   198,\n",
            "         8873, 46481, 16175,   461,   282,   197,  1079,   282, 11797,   198,\n",
            "         1559,  1044, 30102,    81,   197,  1079,   282, 11797,   198,  1681,\n",
            "          325,   479,   874, 30102,    77,   197,  1079,   282, 11797,   198,\n",
            "        16390, 33847,   197,  1079,   282, 11797,   198, 10257,   282,   197,\n",
            "         1079,   282, 11797,   198, 10257,   282,   316,   197,  1079,   282,\n",
            "        11797,   198,  5209,   197,  1079,   282, 11797,   198,  5209, 21885,\n",
            "          321,   197,  1079,   282, 11797,   198,  1525,   416,   197,  1079,\n",
            "          282, 11797,   198, 16390,   197,  1079,   282, 11797,   198,    70,\n",
            "        30570,  9116, 46481,    76,   988, 22825, 33133,    72,  2349,   197,\n",
            "         1079,   282, 11797,   198,  7053,   385,    76,   988,   334,    89,\n",
            "          567,   197,  1079,   282, 11797,   198,    70, 30570])\n",
            "tensor([ 9116, 46481,    76,   988,  6184,   120,    89,   567,   197,  1079,\n",
            "          282, 11797,   198,    70,  9116,   293,   308,  9116,   293,   197,\n",
            "         1079,   282, 11797,   198,    71,  9189,   416,   197,  1079,   282,\n",
            "        11797,   198,  8873, 46481, 16175,   461,   282, 30102,    77,   197,\n",
            "         1079,   282, 11797,   198,  7745,    72, 47594, 46481,   321, 21681,\n",
            "          197,  1079,   282, 11797,   198,  7745,    72,  6184,   100,   282,\n",
            "        30102, 46481,  7617,   283,   197,  1079,   282, 11797,   198,  7745,\n",
            "           72,   308,  9116,    77,  1754,   288,  5329,   320,   197,  1079,\n",
            "          282, 11797,   198,    74,   437,   259,  1096,  1312, 48111,   275,\n",
            "          461, 30102,    77,   197,  1079,   282, 11797,   198,    74,   571,\n",
            "          197,  1079,   282, 11797,   198,   417,  1079,    64,   197,  1079,\n",
            "          282, 11797,   198,    70, 30570,  9116, 46481, 25151,  9116,    89,\n",
            "          197,  1079,   282, 11797,   198,  8873, 46481, 16175])\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [],
      "metadata": {
        "id": "FSJDWc2ajRMX"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# Eğitim için veri koleksiyonunu hazırlama\n",
        "data_collator = DataCollatorForLanguageModeling(\n",
        "    tokenizer=tokenizer,\n",
        "    mlm=False  # Dil modellemesi için maskleme (masked language modeling) kullanmıyoruz\n",
        ")\n"
      ],
      "metadata": {
        "id": "8ulyhm4obt23"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# Eğitim argümanlarını ayarlama\n",
        "training_args = TrainingArguments(\n",
        "    output_dir=\"./fine_tuned_model\",\n",
        "    overwrite_output_dir=True,\n",
        "    num_train_epochs=3,  # Eğitim döngüsü sayısı\n",
        "    per_device_train_batch_size=8,\n",
        "    save_steps=10_000,  # Her bu kadar adımda bir modeli kaydet\n",
        "    save_total_limit=2,  # Kaç model saklanacağını belirle\n",
        "    evaluation_strategy=\"steps\",\n",
        "    eval_steps=10_000,  # Her bu kadar adımda bir değerlendirme yap\n",
        "    remove_unused_columns=False,\n",
        "    push_to_hub=False,\n",
        "    report_to=\"tensorboard\",\n",
        "    load_best_model_at_end=True,\n",
        "    logging_dir=\"./logs\",\n",
        ")\n"
      ],
      "metadata": {
        "id": "9gdTlAiQbzBQ"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# Trainer oluşturma\n",
        "trainer = Trainer(\n",
        "    model=model,\n",
        "    args=training_args,\n",
        "    data_collator=data_collator,\n",
        "    train_dataset=dataset,\n",
        ")"
      ],
      "metadata": {
        "id": "dKdfvo0lc6m5"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "# Fine-tuning işlemini başlatma\n",
        "trainer.train()\n"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 129
        },
        "id": "7kyHyJ1Sc8Me",
        "outputId": "bc430718-85f9-4fbb-c2fd-af9430babff1"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "display_data",
          "data": {
            "text/plain": [
              "<IPython.core.display.HTML object>"
            ],
            "text/html": [
              "\n",
              "    <div>\n",
              "      \n",
              "      <progress value='6' max='6' style='width:300px; height:20px; vertical-align: middle;'></progress>\n",
              "      [6/6 01:01, Epoch 3/3]\n",
              "    </div>\n",
              "    <table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              " <tr style=\"text-align: left;\">\n",
              "      <th>Step</th>\n",
              "      <th>Training Loss</th>\n",
              "      <th>Validation Loss</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "  </tbody>\n",
              "</table><p>"
            ]
          },
          "metadata": {}
        },
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "TrainOutput(global_step=6, training_loss=6.126461664835612, metrics={'train_runtime': 74.419, 'train_samples_per_second': 0.645, 'train_steps_per_second': 0.081, 'total_flos': 3135504384000.0, 'train_loss': 6.126461664835612, 'epoch': 3.0})"
            ]
          },
          "metadata": {},
          "execution_count": 106
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "prompt = \"Sigorta botu musun?\"  # Sigorta botuna sormak istediğiniz soruyu burada belirtin\n",
        "\n",
        "# Prompt'ı tokenize edin\n",
        "input_ids = tokenizer.encode(prompt, return_tensors=\"pt\")\n",
        "\n",
        "# Modeli kullanarak cevap alın\n",
        "with torch.no_grad():\n",
        "    output = model.generate(input_ids, max_length=50, num_return_sequences=1, pad_token_id=tokenizer.eos_token_id)\n",
        "\n",
        "# Cevabı decode edin\n",
        "response = tokenizer.decode(output[0], skip_special_tokens=True)\n",
        "\n",
        "# Cevabı yazdırın\n",
        "print(\"Sigorta Botu Cevabı:\", response)\n",
        "\n",
        "\n"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "M3Iw9pvMoz-O",
        "outputId": "35f4555f-76bf-4c66-decc-3773faa1232b"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Sigorta Botu Cevabı: Sigorta botu musun?\n",
            "Bu sitede yer alan bilgiler, yorum ve tavsiyeler, yorum ve tavsiyeler, yorum ve tavsiyede bulunanların kişisel görüşlerine dayanmaktadır. Bu sitede yer alan bilgiler, yorum ve tavsiyede bulunanların kişisel görüşlerine dayanmaktadır. Bu sitede yer alan bilgiler, yorum\n"
          ]
        }
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "# Sorunuzu veya ifadenizi belirtin\n",
        "soru = \"Sen bir sigorta botu musun?\"\n",
        "\n",
        "# Soruyu tokenize edin\n",
        "input_ids = tokenizer.encode(soru, return_tensors=\"pt\")\n",
        "\n",
        "# Modeli kullanarak cevap üretin\n",
        "output = model.generate(input_ids, max_length=100, num_return_sequences=1, pad_token_id=50256)\n",
        "\n",
        "# Cevabı çıkartın ve ekrana yazdırın\n",
        "cevap = tokenizer.decode(output[0], skip_special_tokens=True)\n",
        "print(cevap)"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "V34dCdnk3DFD",
        "outputId": "883b32e0-06f2-457b-b9c7-3be571b2076c"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu musun? Sen bir sigorta botu\n"
          ]
        }
      ]
    }
  ]
}