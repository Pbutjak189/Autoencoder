# Autoencoder
{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "name": "Autoencoder.ipynb",
      "provenance": [],
      "authorship_tag": "ABX9TyNqAl9x1RdKrm6PrLrssvTc",
      "include_colab_link": true
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
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
        "<a href=\"https://colab.research.google.com/github/GovindSinghShekhawat/NeuaralNetworks/blob/main/Autoencoder.ipynb\" target=\"_parent\"><img src=\"https://colab.research.google.com/assets/colab-badge.svg\" alt=\"Open In Colab\"/></a>"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "XPwrUfVCtRx5"
      },
      "source": [
        "Loading the data"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "EDk720HOn1ks"
      },
      "source": [
        "import os"
      ],
      "execution_count": 1,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "IVgA8gcAoFpg"
      },
      "source": [
        "os.environ[\"CUDA_DEVICE_ORDER\"]=\"PCI_BUS_ID\"\r\n",
        "os.environ[\"CUDA_VISIBLE_DEVICES\"]=\"1\""
      ],
      "execution_count": 2,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "x9T6Hia_odtC"
      },
      "source": [
        "import keras \r\n",
        "from matplotlib import pyplot as plt\r\n",
        "import numpy as np\r\n",
        "import gzip\r\n",
        "%matplotlib inline \r\n",
        "from keras.layers import Input,Conv2D,MaxPooling2D,UpSampling2D\r\n",
        "from keras.models import Model \r\n",
        "from keras.optimizers import RMSprop"
      ],
      "execution_count": 3,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "OG_vu-0CpQ3w"
      },
      "source": [
        "def extract_data(filename, num_images):\r\n",
        "  with gzip.open(filename) as bytestream:\r\n",
        "    bytestream.read(16)\r\n",
        "    buf = bytestream.read(28 * 28 * num_images)\r\n",
        "    data = np.frombuffer(buf, dtype=np.uint8).astype(np.float32)\r\n",
        "    data = data.reshape(num_images, 28, 28)\r\n",
        "    return data"
      ],
      "execution_count": 4,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "5yZkqXWdqUeI"
      },
      "source": [
        "train_data = extract_data('/content/train-images-idx3-ubyte.gz',60000)\r\n",
        "test_data = extract_data('/content/t10k-images-idx3-ubyte.gz',10000)"
      ],
      "execution_count": 8,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "GPgrY7ZmsC1Q"
      },
      "source": [
        "def extract_labels(filename, num_images):\r\n",
        "  with gzip.open(filename) as bytestream:\r\n",
        "    bytestream.read(8)\r\n",
        "    buf = bytestream.read(1 * num_images)\r\n",
        "    labels = np.frombuffer(buf, dtype=np.uint8).astype(np.int64)\r\n",
        "    return labels"
      ],
      "execution_count": 9,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "ONAZzMXwso1Q"
      },
      "source": [
        "train_labels = extract_labels('/content/train-labels-idx1-ubyte.gz',60000)\r\n",
        "test_labels = extract_labels('/content/t10k-labels-idx1-ubyte.gz',10000)"
      ],
      "execution_count": 10,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "QQ18EY4QtaB5"
      },
      "source": [
        "Data Exploration"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "PEq1yltstcye",
        "outputId": "6f09d0c7-386e-4971-ed08-1f960df6df0b"
      },
      "source": [
        "print(\"Training set (images) shape: {shape}\".format(shape=train_data.shape))\r\n",
        "print(\"Test set (images) shape: {shape}\".format(shape=test_data.shape))"
      ],
      "execution_count": 11,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "Training set (images) shape: (60000, 28, 28)\n",
            "Test set (images) shape: (10000, 28, 28)\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "VXff6vBiuaMJ"
      },
      "source": [
        "# Create dictionary of target classes\r\n",
        "label_dict = {\r\n",
        "    0: 'A',\r\n",
        "    1: 'B',\r\n",
        "    2: 'C',\r\n",
        "    3: 'D',\r\n",
        "    4: 'E',\r\n",
        "    5: 'F',\r\n",
        "    6: 'G',\r\n",
        "    7: 'H',\r\n",
        "    8: 'I',\r\n",
        "    9: 'J'\r\n",
        "}"
      ],
      "execution_count": 12,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 208
        },
        "id": "uBmo8UK7vSM2",
        "outputId": "7a39d420-3f25-406b-abed-48aa916d7f04"
      },
      "source": [
        "# Taking a look at couple of images in the dataset\r\n",
        "plt.figure(figsize=[5,5])\r\n",
        "\r\n",
        "# Display the first image in training data\r\n",
        "plt.subplot(121)\r\n",
        "curr_img = np.reshape(train_data[0],(28,28))\r\n",
        "curr_lbl = train_labels[0]\r\n",
        "plt.imshow(curr_img, cmap='gray')\r\n",
        "plt.title(\"(Label: \" + str(label_dict[curr_lbl]) + \")\")\r\n",
        "\r\n",
        "# Display the first image in testing data\r\n",
        "plt.subplot(122)\r\n",
        "curr_img = np.reshape(test_data[0], (28,28))\r\n",
        "curr_lbl = test_labels[0]\r\n",
        "plt.imshow(curr_img, cmap='gray')\r\n",
        "plt.title(\"(Label: \" + str(label_dict[curr_lbl]) + \")\")"
      ],
      "execution_count": 13,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "Text(0.5, 1.0, '(Label: D)')"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 13
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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\n",
            "text/plain": [
              "<Figure size 360x360 with 2 Axes>"
            ]
          },
          "metadata": {
            "tags": [],
            "needs_background": "light"
          }
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "mDUEWxJAxVfn"
      },
      "source": [
        "Data Processing"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "PyMOA5HVxXQp",
        "outputId": "a9d384af-8732-47f8-9270-7686951a93d7"
      },
      "source": [
        "train_data = train_data.reshape(-1,28,28,1)\r\n",
        "test_data = test_data.reshape(-1,28,28,1)\r\n",
        "train_data.shape, test_data.shape"
      ],
      "execution_count": 14,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "((60000, 28, 28, 1), (10000, 28, 28, 1))"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 14
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "B28vlW5dx2_M",
        "outputId": "73df7b3d-ea52-485d-dd72-ae9b78e339ee"
      },
      "source": [
        "train_data.dtype, test_data.dtype"
      ],
      "execution_count": 15,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "(dtype('float32'), dtype('float32'))"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 15
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "RcJEd_7DyAyh",
        "outputId": "ceb85424-b2ac-4171-cb56-9d7fe49c710e"
      },
      "source": [
        "np.max(train_data), np.max(test_data)"
      ],
      "execution_count": 16,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "(255.0, 255.0)"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 16
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "UHzvfgBEyLmt",
        "outputId": "d7148e0b-4288-4dbc-e3eb-4f4c5d82ecb3"
      },
      "source": [
        "train_data = train_data / np.max(train_data)\r\n",
        "test_data = test_data / np.max(test_data)\r\n",
        "np.max(train_data), np.max(test_data)"
      ],
      "execution_count": 18,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "(1.0, 1.0)"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 18
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "n-n6stkQyjTM"
      },
      "source": [
        "from sklearn.model_selection import train_test_split\r\n",
        "train_X,valid_X,train_ground,valid_ground = train_test_split(train_data,\r\n",
        "                                train_data,test_size=0.2,random_state=13)"
      ],
      "execution_count": 20,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "lJ2gGrlrzL3m"
      },
      "source": [
        "The Convolutional Autoencoder"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "46wo8efSzPog"
      },
      "source": [
        "batch_size = 128\r\n",
        "epochs = 50\r\n",
        "inChannel = 1\r\n",
        "x, y = 28, 28\r\n",
        "input_img = Input(shape = (x, y, inChannel))"
      ],
      "execution_count": 21,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "1Sm2ZI3pzqzv"
      },
      "source": [
        "def autoencoder(input_img):\r\n",
        "  #encoder\r\n",
        "  #input = 28 x 28 x 1 (wide and thin)\r\n",
        "  conv1 = Conv2D(32, (3,3), activation='relu', padding='same')(input_img) #28x28x32\r\n",
        "  pool1 = MaxPooling2D(pool_size=(2,2))(conv1) #14x14x32\r\n",
        "  conv2 = Conv2D(64, (3,3), activation ='relu',padding='same')(pool1) #14x14x64\r\n",
        "  pool2 = MaxPooling2D(pool_size=(2,2))(conv2) #7x7x64\r\n",
        "  conv3 = Conv2D(128, (3,3), activation='relu', padding='same')(pool2) #7x7x128(narrow and thick)\r\n",
        "\r\n",
        "  #decoder\r\n",
        "  conv4 = Conv2D(128, (3,3), activation='relu', padding='same')(conv3) #7x7x128\r\n",
        "  up1 = UpSampling2D((2,2))(conv4) #14x14x128\r\n",
        "  conv5 = Conv2D(64, (3,3), activation='relu', padding='same')(up1) #14x14x64\r\n",
        "  up2 = UpSampling2D((2,2))(conv5) #28x28x64\r\n",
        "  decoded = Conv2D(1, (3,3), activation='sigmoid', padding='same')(up2) #28x28x1\r\n",
        "  return decoded"
      ],
      "execution_count": 27,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "9JxAK3CS3jK4"
      },
      "source": [
        "autoencoder = Model(input_img, autoencoder(input_img))\r\n",
        "autoencoder.compile(loss='mean_squared_error', optimizer = RMSprop())"
      ],
      "execution_count": 28,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "_hxXWdCG4YOE",
        "outputId": "ffbc7d83-db00-4c76-be02-64a09defaa1f"
      },
      "source": [
        "autoencoder.summary()"
      ],
      "execution_count": 29,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "Model: \"model\"\n",
            "_________________________________________________________________\n",
            "Layer (type)                 Output Shape              Param #   \n",
            "=================================================================\n",
            "input_1 (InputLayer)         [(None, 28, 28, 1)]       0         \n",
            "_________________________________________________________________\n",
            "conv2d_13 (Conv2D)           (None, 28, 28, 32)        320       \n",
            "_________________________________________________________________\n",
            "max_pooling2d_6 (MaxPooling2 (None, 14, 14, 32)        0         \n",
            "_________________________________________________________________\n",
            "conv2d_14 (Conv2D)           (None, 14, 14, 64)        18496     \n",
            "_________________________________________________________________\n",
            "max_pooling2d_7 (MaxPooling2 (None, 7, 7, 64)          0         \n",
            "_________________________________________________________________\n",
            "conv2d_15 (Conv2D)           (None, 7, 7, 128)         73856     \n",
            "_________________________________________________________________\n",
            "conv2d_16 (Conv2D)           (None, 7, 7, 128)         147584    \n",
            "_________________________________________________________________\n",
            "up_sampling2d_3 (UpSampling2 (None, 14, 14, 128)       0         \n",
            "_________________________________________________________________\n",
            "conv2d_17 (Conv2D)           (None, 14, 14, 64)        73792     \n",
            "_________________________________________________________________\n",
            "up_sampling2d_4 (UpSampling2 (None, 28, 28, 64)        0         \n",
            "_________________________________________________________________\n",
            "conv2d_18 (Conv2D)           (None, 28, 28, 1)         577       \n",
            "=================================================================\n",
            "Total params: 314,625\n",
            "Trainable params: 314,625\n",
            "Non-trainable params: 0\n",
            "_________________________________________________________________\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "gKJEjAIQ4gc8"
      },
      "source": [
        "Train the model"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "J6hFEo_f4i5x",
        "outputId": "f820f9c7-cb0f-48fc-9871-adc7b66295b2"
      },
      "source": [
        "autoencoder_train = autoencoder.fit(train_X, train_ground, batch_size=batch_size,\r\n",
        "                    epochs=epochs, verbose=1, validation_data=(valid_X, valid_ground))"
      ],
      "execution_count": 30,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "Epoch 1/50\n",
            "375/375 [==============================] - 359s 953ms/step - loss: 0.0674 - val_loss: 0.0113\n",
            "Epoch 2/50\n",
            "375/375 [==============================] - 357s 951ms/step - loss: 0.0112 - val_loss: 0.0090\n",
            "Epoch 3/50\n",
            "375/375 [==============================] - 360s 961ms/step - loss: 0.0076 - val_loss: 0.0063\n",
            "Epoch 4/50\n",
            "375/375 [==============================] - 359s 958ms/step - loss: 0.0060 - val_loss: 0.0045\n",
            "Epoch 5/50\n",
            "375/375 [==============================] - 360s 959ms/step - loss: 0.0051 - val_loss: 0.0039\n",
            "Epoch 6/50\n",
            "375/375 [==============================] - 359s 957ms/step - loss: 0.0044 - val_loss: 0.0043\n",
            "Epoch 7/50\n",
            "375/375 [==============================] - 360s 959ms/step - loss: 0.0039 - val_loss: 0.0039\n",
            "Epoch 8/50\n",
            "375/375 [==============================] - 362s 966ms/step - loss: 0.0036 - val_loss: 0.0029\n",
            "Epoch 9/50\n",
            "375/375 [==============================] - 359s 956ms/step - loss: 0.0033 - val_loss: 0.0033\n",
            "Epoch 10/50\n",
            "375/375 [==============================] - 361s 964ms/step - loss: 0.0030 - val_loss: 0.0028\n",
            "Epoch 11/50\n",
            "375/375 [==============================] - 359s 958ms/step - loss: 0.0029 - val_loss: 0.0032\n",
            "Epoch 12/50\n",
            "375/375 [==============================] - 358s 955ms/step - loss: 0.0028 - val_loss: 0.0029\n",
            "Epoch 13/50\n",
            "375/375 [==============================] - 360s 959ms/step - loss: 0.0027 - val_loss: 0.0032\n",
            "Epoch 14/50\n",
            "375/375 [==============================] - 361s 964ms/step - loss: 0.0026 - val_loss: 0.0024\n",
            "Epoch 15/50\n",
            "375/375 [==============================] - 360s 959ms/step - loss: 0.0024 - val_loss: 0.0026\n",
            "Epoch 16/50\n",
            "375/375 [==============================] - 359s 958ms/step - loss: 0.0024 - val_loss: 0.0024\n",
            "Epoch 17/50\n",
            "375/375 [==============================] - 362s 966ms/step - loss: 0.0023 - val_loss: 0.0025\n",
            "Epoch 18/50\n",
            "375/375 [==============================] - 361s 963ms/step - loss: 0.0023 - val_loss: 0.0021\n",
            "Epoch 19/50\n",
            "375/375 [==============================] - 361s 964ms/step - loss: 0.0022 - val_loss: 0.0019\n",
            "Epoch 20/50\n",
            "375/375 [==============================] - 363s 968ms/step - loss: 0.0021 - val_loss: 0.0019\n",
            "Epoch 21/50\n",
            "375/375 [==============================] - 361s 963ms/step - loss: 0.0021 - val_loss: 0.0019\n",
            "Epoch 22/50\n",
            "375/375 [==============================] - 364s 970ms/step - loss: 0.0020 - val_loss: 0.0019\n",
            "Epoch 23/50\n",
            "375/375 [==============================] - 364s 970ms/step - loss: 0.0020 - val_loss: 0.0018\n",
            "Epoch 24/50\n",
            "375/375 [==============================] - 362s 967ms/step - loss: 0.0019 - val_loss: 0.0021\n",
            "Epoch 25/50\n",
            "375/375 [==============================] - 364s 972ms/step - loss: 0.0019 - val_loss: 0.0019\n",
            "Epoch 26/50\n",
            "375/375 [==============================] - 362s 967ms/step - loss: 0.0019 - val_loss: 0.0020\n",
            "Epoch 27/50\n",
            "375/375 [==============================] - 362s 966ms/step - loss: 0.0019 - val_loss: 0.0018\n",
            "Epoch 28/50\n",
            "375/375 [==============================] - 364s 969ms/step - loss: 0.0018 - val_loss: 0.0018\n",
            "Epoch 29/50\n",
            "375/375 [==============================] - 362s 965ms/step - loss: 0.0018 - val_loss: 0.0020\n",
            "Epoch 30/50\n",
            "375/375 [==============================] - 364s 971ms/step - loss: 0.0018 - val_loss: 0.0022\n",
            "Epoch 31/50\n",
            "375/375 [==============================] - 364s 972ms/step - loss: 0.0018 - val_loss: 0.0020\n",
            "Epoch 32/50\n",
            "375/375 [==============================] - 364s 972ms/step - loss: 0.0017 - val_loss: 0.0020\n",
            "Epoch 33/50\n",
            "375/375 [==============================] - 367s 979ms/step - loss: 0.0017 - val_loss: 0.0017\n",
            "Epoch 34/50\n",
            "375/375 [==============================] - 363s 968ms/step - loss: 0.0017 - val_loss: 0.0016\n",
            "Epoch 35/50\n",
            "375/375 [==============================] - 364s 970ms/step - loss: 0.0017 - val_loss: 0.0021\n",
            "Epoch 36/50\n",
            "375/375 [==============================] - 364s 971ms/step - loss: 0.0017 - val_loss: 0.0017\n",
            "Epoch 37/50\n",
            "375/375 [==============================] - 364s 972ms/step - loss: 0.0016 - val_loss: 0.0017\n",
            "Epoch 38/50\n",
            "375/375 [==============================] - 367s 978ms/step - loss: 0.0016 - val_loss: 0.0016\n",
            "Epoch 39/50\n",
            "375/375 [==============================] - 363s 969ms/step - loss: 0.0016 - val_loss: 0.0015\n",
            "Epoch 40/50\n",
            "375/375 [==============================] - 364s 970ms/step - loss: 0.0016 - val_loss: 0.0016\n",
            "Epoch 41/50\n",
            "375/375 [==============================] - 364s 972ms/step - loss: 0.0016 - val_loss: 0.0017\n",
            "Epoch 42/50\n",
            "375/375 [==============================] - 365s 975ms/step - loss: 0.0016 - val_loss: 0.0014\n",
            "Epoch 43/50\n",
            "375/375 [==============================] - 365s 974ms/step - loss: 0.0016 - val_loss: 0.0015\n",
            "Epoch 44/50\n",
            "375/375 [==============================] - 365s 972ms/step - loss: 0.0016 - val_loss: 0.0016\n",
            "Epoch 45/50\n",
            "375/375 [==============================] - 364s 971ms/step - loss: 0.0015 - val_loss: 0.0016\n",
            "Epoch 46/50\n",
            "375/375 [==============================] - 365s 973ms/step - loss: 0.0015 - val_loss: 0.0017\n",
            "Epoch 47/50\n",
            "375/375 [==============================] - 366s 976ms/step - loss: 0.0015 - val_loss: 0.0016\n",
            "Epoch 48/50\n",
            "375/375 [==============================] - 363s 967ms/step - loss: 0.0015 - val_loss: 0.0016\n",
            "Epoch 49/50\n",
            "375/375 [==============================] - 362s 966ms/step - loss: 0.0015 - val_loss: 0.0015\n",
            "Epoch 50/50\n",
            "375/375 [==============================] - 359s 958ms/step - loss: 0.0015 - val_loss: 0.0018\n"
          ],
          "name": "stdout"
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "VAmjXUr5TRNy"
      },
      "source": [
        "Training vs Validation Loss Plot"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 281
        },
        "id": "mu5Rx2TmTPIz",
        "outputId": "75a8c9de-89ec-4dd5-a071-9f1deb4738d2"
      },
      "source": [
        "loss = autoencoder_train.history['loss']\r\n",
        "val_loss = autoencoder_train.history['val_loss']\r\n",
        "epochs = range(epochs)\r\n",
        "plt.figure()\r\n",
        "plt.plot(epochs, loss, 'bo', label='Training loss')\r\n",
        "plt.plot(epochs, val_loss, 'b', label='Validation loss')\r\n",
        "plt.title('Training and validation loss')\r\n",
        "plt.legend()\r\n",
        "plt.show()"
      ],
      "execution_count": 31,
      "outputs": [
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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EA4++GAef/zxhv2BVaNVPw+g/ELvFVck3T69eycHf18ANmt41V1za+j/ubKyMp588knatGnDxo0befzxxykqKuKhhx7iu9/9Lr/5zW922GblypU88sgjbNq0iQMPPJDzzz9/h/Hyzz77LCtWrGDfffdl5MiRPPHEE5SUlPD1r3+dxx57jH79+jFhwoQa63f11VczdOhQ7r33XhYvXsykSZNYvnw5c+bMYe7cuYwcOZLNmzfToUMH5s2bx5e//GWuuOIKPvnkE7ZU/iE2olYdACD5w/MB36zxNeU1t9NPP502bdoAsGHDBiZPnszf/vY3JLF169a825x88sm0b9+e9u3bs+eee/LWW2/Rs2fPCnlGjBixPW3IkCGsWbOGzp07s99++20fWz9hwgTmzZu3Q/m5lixZsj0IHX/88axfv56NGzcycuRILr74YiZOnMhpp51Gz549GT58OFOnTmXr1q2ceuqpDBkypF4/m9po1V1AZtZ0mvKa26677rr985VXXslxxx3HCy+8wH333VflOPj27dtv/9ymTZu81w8KyVMfM2bM4KabbuKDDz5g5MiRrFy5kmOOOYbHHnuMHj16MGXKFH75y1826D6r4wBgZg2iua65bdiwgR49kvkpb7311gYv/8ADD+SVV15hzZo1ANx55501bnP00UczP7348eijj9KtWzd22203Xn75ZQYNGsRll13G8OHDWblyJWvXrmWvvfbi3HPP5ZxzzuGZZ55p8DZUxQHAzBrExIkwbx706QNS8j5vXuN3wV566aVcfvnlDB06tMG/sQN07NiRG2+8kVGjRjFs2DC6dOlC165dq91m5syZLFu2jMGDBzNjxgxuuy2ZLf+6667j4IMPZvDgwbRt25bRo0fz6KOPcsghhzB06FDuvPNOLrroogZvQ1V2qmcCl5SUhB8IY9Z0XnrpJT7/+c83dzWa3ebNm+ncuTMRwQUXXED//v2ZPn16c1crr3y/M0nL8j110WcAZmY1+PnPf86QIUMYOHAgGzZs4Otf/3pzV6lBtPpRQGZm9TV9+vQW+42/PnwGYGaWUQ4AZmYZ5QBgZpZRDgBmZhnlAGBmLdZxxx3HokWLKqRdd911nH/++VVuc+yxx1I+XPykk07i/fff3yHPzJkzmTNnTrX7vvfee3nxxRe3L1911VU89NBDtal+Xi1p2mgHADNrsSZM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\n",
            "text/plain": [
              "<Figure size 432x288 with 1 Axes>"
            ]
          },
          "metadata": {
            "tags": [],
            "needs_background": "light"
          }
        }
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "H8DOSGBr-S7P"
      },
      "source": [
        "Predicting on test data"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "tKpS2cbK-VXM",
        "outputId": "118bf7ef-3fa8-4677-a2cd-e0cc58716065"
      },
      "source": [
        "pred = autoencoder.predict(test_data)\r\n",
        "pred.shape"
      ],
      "execution_count": 32,
      "outputs": [
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "(10000, 28, 28, 1)"
            ]
          },
          "metadata": {
            "tags": []
          },
          "execution_count": 32
        }
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 202
        },
        "id": "8O-hbBIj-oFi",
        "outputId": "67177228-a40e-4bc8-bbd4-b6e6b3c4831b"
      },
      "source": [
        "plt.figure(figsize=(20,4))\r\n",
        "print(\"Test Images\")\r\n",
        "for i in range(10):\r\n",
        "  plt.subplot(2, 10, i+1)\r\n",
        "  plt.imshow(test_data[i, ...,0], cmap='gray')\r\n",
        "  curr_lbl = test_labels[i]\r\n",
        "  plt.title(\"(Label: \" + str(label_dict[curr_lbl]) + \")\")\r\n",
        "plt.show()\r\n",
        "plt.figure(figsize=(20,4))\r\n",
        "print(\"Reconstruction of Test Images\")\r\n",
        "for i in range(10):\r\n",
        "  plt.subplot(2, 10, i+1)\r\n",
        "  plt.imshow(pred[i, ..., 0], cmap = 'gray')\r\n",
        "plt.show()"
      ],
      "execution_count": 35,
      "outputs": [
        {
          "output_type": "stream",
          "text": [
            "Test Images\n"
          ],
          "name": "stdout"
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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z04BKynkOLU1lN9bujYBwBNPPKFte+6SvPrqq9q27yPkOY+Ke5fEdluTfTzxxBO1Ld15AXM9iEpq+oricuGLuvua636ySZMm2pZrD2C6brrmN0KiiLxm7XVR4lr7ZBv2XJZPXOt4+nWu5lZ77pPznQzXAJjnWfYn23Mnv6s9B8kwD3INss+D7NPf//53o+zSSy/Vtky3breRTf/t/srzVrVqVaOsT58+2r7pppt82+SOH0IIIYQQQgghhJCEwgc/hBBCCCGEEEIIIQmFD34IIYQQQgghhBBCEgpj/OQA6U/oig8j08Pdd999Rpn043P5hZ988snattMPu9KGEzcuf13JFVdcoe0JEyb41gsab8muJ19HPQZARXF9dxeulPdB/ZVnzJjhWxbVGBqVK1f2TWndpUsXbRcyps/27du1LWMnxeFadvnzh0mlSpV0fBc7vpQ8T664Afvuu69v+zt27NC2HUPM71jZ6uP6nF+cA3udkj73LqIaq8oVx2TYsGFGWbt27bTtFxsQAO65554wu5hXXNfEF198oe1PPvkkqzaihry27TEbp++RCbVq1dL2PvvsY5R9/fXX2maMHxJnXOPXtVZLonQPGRXsGLeSMOYIqY0dZ0euQevXr9e2vQa7fo/861//0raM8ZPr+c1uXx7bBXf8EEIIIYQQQgghhCSUPT74UUr9Qym1Sik1W7xXXyk1QSk1P/V/+X92JpGBOiaC1tQw/nAsJgKOxQTAsZgIOBYTAMdiIuBYTAAci8kmyF72MQD+DuCf4r0bAUz0PO8+pdSNqdd/DL978ce1NXD8+PHatrdySzcD13axDh06aNve7i9TVCPBOmbrIiTruVINyq2Zf/7zn416xx13nLZfeumlQMd1sYet4GsAXIQEagj4p5cE3JpKdwkXrjY+//zzrD6XJWMQwlisX78+LrzwQgDQad3TSPeefv36+baRbap6P/c3OR8BwNtvv63td955R9u1a9c26sVwy39oY7FWrVro1KkTAGDLli1G2dSpU30/J1OD+7n8AabrlCtVunSdki7EAHDooYdqu1mzZtqWLh4AsGjRIm2//vrrRtn06dO1vXPnTt9+bNy40bdMYqdGz5IxCGldTG8ht7fy9+zZU9sy7Stgngfpqm2vJR999JG2XWtVFHGNbXm9ZNsGCrAu2u4Csn8uV46jjjpK26WlpUbZ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\n",
            "text/plain": [
              "<Figure size 1440x288 with 10 Axes>"
            ]
          },
          "metadata": {
            "tags": [],
            "needs_background": "light"
          }
        },
        {
          "output_type": "stream",
          "text": [
            "Reconstruction of Test Images\n"
          ],
          "name": "stdout"
        },
        {
          "output_type": "display_data",
          "data": {
            "image/png": "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\n",
            "text/plain": [
              "<Figure size 1440x288 with 10 Axes>"
            ]
          },
          "metadata": {
            "tags": [],
            "needs_background": "light"
          }
        }
      ]
    }
  ]
}
