# 第十七章。网络驱动程序，第二部分：数据包接收和传输

![无标题图片](http://atomoreilly.com/source/nostarch/images/1137497.png.jpg)

本章探讨了 `em(4)` 的数据包接收和传输组件。不出所料，`em(4)` 使用 mbuf 和 MSI 进行数据包接收和传输。

# 数据包接收

当一个接口接收到一个数据包时，它会发送一个中断。自然地，这会导致其中断处理程序执行。例如，以下是在 `em(4)` 中执行的内容：

```
static void
em_msix_rx(void *arg)
{
        struct rx_ring *rxr = arg;
        struct adapter *adapter = rxr->adapter;
        bool more;

        ++rxr->rx_irq;

        more = em_rxeof(rxr, adapter->rx_process_limit, NULL);
        if (more)
                taskqueue_enqueue(rxr->tq, &rxr->rx_task);
        else
                E1000_WRITE_REG(&adapter->hw, E1000_IMS, rxr->ims);
}
```

此函数接收一个 ![环缓冲区指针图片](http://atomoreilly.com/source/nostarch/images/1137499.png) 指向包含一个或多个接收到的数据包的环缓冲区的指针，并调用 ![em_rxeof 图片](http://atomoreilly.com/source/nostarch/images/1137501.png) `em_rxeof` 来处理这些数据包。如果有超过 ![rx_process_limit 图片](http://atomoreilly.com/source/nostarch/images/1137503.png) `rx_process_limit` 个数据包，则将 `task` 结构排队；否则，此中断 ![重新启用图片](http://atomoreilly.com/source/nostarch/images/1137505.png) 重新启用。我将在 em_handle_rx 函数 中讨论 `task` 结构及其相关函数。

## em_rxeof 函数

如前所述，`em_rxeof` 处理接收到的数据包。其函数定义如下，但由于此函数相当长且复杂，我将分部分介绍。以下是第一部分：

```
static bool
em_rxeof(struct rx_ring *rxr, int count, int *done)
{
        struct adapter *adapter = rxr->adapter;
        struct ifnet *ifp = adapter->ifp;
        struct e1000_rx_desc *cur;
        struct mbuf *mp, *sendmp;
        u8 status = 0;
        u16 len;
        int i, processed, rxdone = 0;
        bool eop;

        EM_RX_LOCK(rxr);

      for (i = rxr->next_to_check, processed = 0; count != 0; ) {
              if ((ifp->if_drv_flags & IFF_DRV_RUNNING) == 0)
                        break;

              bus_dmamap_sync(rxr->rxdma.dma_tag, rxr->rxdma.dma_map,
                    BUS_DMASYNC_POSTREAD);

                mp = sendmp = NULL;
                cur = &rxr->rx_base[i];
                status = cur->status;
                if ((status & E1000_RXD_STAT_DD) == 0)
                        break;
                len = le16toh(cur->length);
                eop = (status & E1000_RXD_STAT_EOP) != 0;

                if ((cur->errors & E1000_RXD_ERR_FRAME_ERR_MASK) ||
                    (rxr->discard == TRUE)) {
                        ++ifp->if_ierrors;
                        ++rxr->rx_discarded;
                        if (!eop)
                                rxr->discard = TRUE;
                        else
                                rxr->discard = FALSE;
                      em_rx_discard(rxr, i);
                        goto next_desc;
                }
...
```

此函数的执行主要在一个 ![for 循环图片](http://atomoreilly.com/source/nostarch/images/1137499.png) `for` 循环中进行。此循环首先 ![验证接口图片](http://atomoreilly.com/source/nostarch/images/1137501.png) 验证接口是否已启动并运行。然后它 ![同步 DMA 缓冲区图片](http://atomoreilly.com/source/nostarch/images/1137505.png) 同步当前加载在 ![DMA 映射图片](http://atomoreilly.com/source/nostarch/images/1137507.png) `rxr->rxdma.dma_map` 中的 DMA 缓冲区，该缓冲区是 ![rx_base 图片](http://atomoreilly.com/source/nostarch/images/1137509.png) `rxr->rx_base`。

缓冲区 ![缓冲区图片](http://atomoreilly.com/source/nostarch/images/1137509.png) `rxr->rx_base[i]` 包含一个描述符，用于描述接收到的数据包。当一个数据包跨越多个 mbuf 时，`rxr->rx_base[i]` 描述链中的其中一个 mbuf。

如果 `rxr->rx_base[i]` 缺少 ![接收描述符状态图片](http://atomoreilly.com/source/nostarch/images/1137511.png) `E1000_RXD_STAT_DD` 标志，则 `for` 循环退出。（`E1000_RXD_STAT_DD` 标志代表 *接收描述符状态：描述符完成*。我们很快就会看到其效果。）

如果 `rxr->rx_base[i]` 描述链中的 ![最后一个 mbuf 图片](http://atomoreilly.com/source/nostarch/images/1137513.png) 最后一个 mbuf，布尔变量 `eop`（代表 *数据包结束*）被设置为 `TRUE`。（不用说，当一个数据包只需要一个 mbuf 时，该 mbuf 仍然是链中的最后一个 mbuf。）

如果由`rxr->rx_base[i]`描述的包包含任何![错误](http://atomoreilly.com/source/nostarch/images/1137515.png)，则![丢弃](http://atomoreilly.com/source/nostarch/images/1137517.png)。请注意，在这里我使用的是单词*包*，而不是*mbuf*，因为包中的每个 mbuf 都被丢弃。

现在，让我们看看`em_rxeof`的下一部分：

```
...
              mp = rxr->rx_buffers[i].m_head;
                mp->m_len = len;
                rxr->rx_buffers[i].m_head = NULL;

              if (rxr->fmp == NULL) {
                        mp->m_pkthdr.len = len;
                      rxr->fmp = rxr->lmp = mp;
                } else {
                        mp->m_flags &= ˜M_PKTHDR;
                      rxr->lmp->m_next = mp;
                      rxr->lmp = mp;
                        rxr->fmp->m_pkthdr.len += len;
                }
...
```

在这里，![rxr->fmp](http://atomoreilly.com/source/nostarch/images/1137505.png)和![rxr->lmp](http://atomoreilly.com/source/nostarch/images/1137507.png)指向链中的第一个和最后一个 mbuf，![mp](http://atomoreilly.com/source/nostarch/images/1137499.png)是`rxr->rx_base[i]`描述的 mbuf，![len](http://atomoreilly.com/source/nostarch/images/1137501.png)是`mp`的长度。

因此，这部分只是![识别](http://atomoreilly.com/source/nostarch/images/1137503.png)是否`mp`是链中的第一个 mbuf。如果不是，则`mp`![被链接](http://atomoreilly.com/source/nostarch/images/1137509.png)![到链中](http://atomoreilly.com/source/nostarch/images/1137511.png)。

这里是`em_rxeof`的下一部分：

```
...
                 if (eop) {
                          --count;
                        sendmp = rxr->fmp;
                            sendmp->m_pkthdr.rcvif = ifp;
                          ++ifp->if_ipackets;
                        em_receive_checksum(cur, sendmp);
 #ifndef __NO_STRICT_ALIGNMENT
                        if (adapter->max_frame_size >
                              (MCLBYTES - ETHER_ALIGN) &&
                            em_fixup_rx(rxr) != 0)
                                  goto skip;
  #endif
                          if (status & E1000_RXD_STAT_VP) {
                                  sendmp->m_pkthdr.ether_vtag =
                                      le16toh(cur->special) &
                                      E1000_RXD_SPC_VLAN_MASK;
                                  sendmp->m_flags |= M_VLANTAG;
                          }
  #ifndef __NO_STRICT_ALIGNMENT
  skip:
  #endif
                        rxr->fmp = rxr->lmp = NULL;
                  }
  ...
```

如果`mp`是链中的![最后一个 mbuf](http://atomoreilly.com/source/nostarch/images/1137499.png)，则![sendmp](http://atomoreilly.com/source/nostarch/images/1137501.png)被设置为链中的![第一个 mbuf](http://atomoreilly.com/source/nostarch/images/1137503.png)，并且验证了头部校验和。

如果我们的架构需要![严格对齐](http://atomoreilly.com/source/nostarch/images/1137507.png)并且启用了![巨帧](http://atomoreilly.com/source/nostarch/images/1137509.png)，则`em_rxeof`![对齐](http://atomoreilly.com/source/nostarch/images/1137511.png) mbuf 链。 (巨帧是包含超过 1500 字节数据的以太网数据包。)

这一部分通过设置![rxr->fmp](http://atomoreilly.com/source/nostarch/images/1137513.png)和![rxr->lmp](http://atomoreilly.com/source/nostarch/images/1137515.png)为![NULL](http://atomoreilly.com/source/nostarch/images/1137517.png)来结束。以下是`em_rxeof`的下一部分：

```
...
next_desc:
                cur->status = 0;
                ++rxdone;
                ++processed;

                if (++i == adapter->num_rx_desc)
                        i = 0;

              if (sendmp != NULL) {
                        rxr->next_to_check = i;
                        EM_RX_UNLOCK(rxr);
                      (*ifp->if_input)(ifp, sendmp);
                        EM_RX_LOCK(rxr);
                        i = rxr->next_to_check;
                }

                if (processed == 8) {
                      em_refresh_mbufs(rxr, i);
                        processed = 0;
                }
        }                                      /* The end of the for loop. */
...
```

在这里，`i`![递增](http://atomoreilly.com/source/nostarch/images/1137499.png)以便`em_rxeof`可以到达环中的下一个 mbuf。然后，如果`sendmp`指向一个 mbuf 链，则执行`em(4)`的输入例程![来发送](http://atomoreilly.com/source/nostarch/images/1137503.png)该链到上层。之后，为`em(4)`![分配](http://atomoreilly.com/source/nostarch/images/1137507.png)新的 mbuf。

### 注意

当将 mbuf 链发送到上层时，驱动程序不得再访问这些 mbuf。从所有目的和用途来看，这些 mbuf 已经被释放。

总结来说，这个 `for` 循环只是将接收到的数据包中的每个 mbuf 链接起来，并将其发送到上层。这个过程会一直持续，直到处理完环中的每个数据包或达到 `rx_process_limit`（`rx_process_limit` 在 数据包接收 中描述过）。

这是 `em_rxeof` 的最后部分：

```
...
        if (e1000_rx_unrefreshed(rxr))
                em_refresh_mbufs(rxr, i);

        rxr->next_to_check = i;
        if (done != NULL)
                *done = rxdone;
        EM_RX_UNLOCK(rxr);

      return ((status & E1000_RXD_STAT_DD) ? TRUE : FALSE);
}
```

如果还有更多数据包需要处理，`em_rxeof` 会返回 `TRUE`。

## em_handle_rx 函数

回想一下，当 `em_rxeof` 返回 TRUE 时，`em_msix_rx` 会将一个任务结构排队（`em_msix_rx` 在 数据包接收 中讨论过）。

这是那个 `task` 结构的函数：

```
static void
em_handle_rx(void *context, int pending)
{
        struct rx_ring *rxr = context;
        struct adapter *adapter = rxr->adapter;
        bool more;

        more = em_rxeof(rxr, adapter->rx_process_limit, NULL);
        if (more)
                taskqueue_enqueue(rxr->tq, &rxr->rx_task);
        else
                E1000_WRITE_REG(&adapter->hw, E1000_IMS, rxr->ims);
}
```

此函数几乎与 `em_msix_rx` 相同。当有更多数据包需要处理时，`em_rxeof` 会被再次调用。

# 数据包传输

为了传输一个数据包，网络栈会调用驱动程序的输出例程。所有输出例程都以调用其接口的传输或启动例程结束。以下是 `em(4)` 的启动例程：

```
static void
em_start(struct ifnet *ifp)
{
        struct adapter *adapter = ifp->if_softc;
        struct tx_ring *txr = adapter->tx_rings;

        if (ifp->if_drv_flags & IFF_DRV_RUNNING) {
              EM_TX_LOCK(txr);
              em_start_locked(ifp, txr);
                EM_TX_UNLOCK(txr);
        }
}
```

此启动例程会获取一个锁，然后调用 `em_start_locked`。

## em_start_locked 函数

`em_start_locked` 函数的定义如下：

```
static void
em_start_locked(struct ifnet *ifp, struct tx_ring *txr)
{
        struct adapter *adapter = ifp->if_softc;
        struct mbuf *m_head;

        EM_TX_LOCK_ASSERT(txr);

        if ((ifp->if_drv_flags & (IFF_DRV_RUNNING | IFF_DRV_OACTIVE)) !=
            IFF_DRV_RUNNING)
                return;

        if (!adapter->link_active)
                return;

      while (!IFQ_DRV_IS_EMPTY(&ifp->if_snd)) {
              if (txr->tx_avail <= EM_TX_CLEANUP_THRESHOLD)
                      em_txeof(txr);

              if (txr->tx_avail < EM_MAX_SCATTER) {
                      ifp->if_drv_flags |= IFF_DRV_OACTIVE;
                        break;
                }

              IFQ_DRV_DEQUEUE(&ifp->if_snd, m_head);
                if (m_head == NULL)
                        break;

                if (em_xmit(txr, &m_head)) {
                        if (m_head == NULL)
                                break;
                        ifp->if_drv_flags |= IFF_DRV_OACTIVE;
                        IFQ_DRV_PREPEND(&ifp->if_snd, m_head);
                        break;
                }

                ETHER_BPF_MTAP(ifp, m_head);

                txr->watchdog_time = ticks;
                txr->queue_status = EM_QUEUE_WORKING;
        }
}
```

此函数从 `em(4)` 的发送队列中移除一个 mbuf 并将其传输到接口。这个过程会一直重复，直到发送队列为空。（发送队列，如第十六章所述，由输出例程填充。）

### 注意

实际将 mbuf 传输到接口的 `em_xmit` 函数在本书中没有详细说明，因为其长度较长。尽管如此，它相当直接，所以你不会遇到任何麻烦。

如果可用的传输描述符数量小于或等于 `EM_TX_CLEANUP_THRESHOLD`，则会调用 `em_txeof` 来回收已使用的描述符。（传输描述符描述一个出站数据包。如果一个数据包跨越多个 mbuf，则传输描述符描述链中的一个 mbuf。）

如果可用的传输描述符数量小于 `EM_MAX_SCATTER`，则传输会被停止。

## em_txeof 函数

`em_txeof` 函数遍历传输描述符，并为已传输的数据包释放 mbufs。其函数定义如下，但由于这个函数相当长且复杂，我将分部分介绍。以下是第一部分：

```
static bool
em_txeof(struct tx_ring *txr)
{
        struct adapter *adapter = txr->adapter;
        struct ifnet *ifp = adapter->ifp;
        struct e1000_tx_desc *tx_desc, *eop_desc;
        struct em_buffer *tx_buffer;
        int processed, first, last, done;

        EM_TX_LOCK_ASSERT(txr);

        if (txr->tx_avail == adapter->num_tx_desc) {
                txr->queue_status = EM_QUEUE_IDLE;
                return (FALSE);
        }

        processed = 0;
      first = txr->next_to_clean;
      tx_desc = &txr->tx_base[first];
      tx_buffer = &txr->tx_buffers[first];
      last = tx_buffer->next_eop;
        eop_desc = &txr->tx_base[last];

        if (++last == adapter->num_tx_desc)
                last = 0;
      done = last;
...
```

在这里，![](img/httpatomoreillycomsourcenostarchimages1137499.png) `first` 是包含传出数据包的链中的第一个 mbuf，![](img/httpatomoreillycomsourcenostarchimages1137505.png) `last` 是该链中的最后一个 mbuf，而 ![](img/httpatomoreillycomsourcenostarchimages1137507.png) `done` 是 `last` 之后的 mbuf。

### 注意

回想一下，传输描述符以及随后的 mbufs 都存储在一个环形缓冲区中。

变量 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) `tx_desc` 和 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) `tx_buffer` 是传输描述符及其相关 mbuf 的临时变量。

现在我们来看 `em_txeof` 的下一部分：

```
...
        bus_dmamap_sync(txr->txdma.dma_tag, txr->txdma.dma_map,
            BUS_DMASYNC_POSTREAD);

      while (eop_desc->upper.fields.status & E1000_TXD_STAT_DD) {
              while (first != done) {
                      tx_desc->upper.data = 0;
                        tx_desc->lower.data = 0;
                        tx_desc->buffer_addr = 0;
                        ++txr->tx_avail;
                        ++processed;

                        if (tx_buffer->m_head) {
                                bus_dmamap_unload(txr->txtag,
                                    tx_buffer->map);
                              m_freem(tx_buffer->m_head);
                                tx_buffer->m_head = NULL;
                        }

                        tx_buffer->next_eop = −1;
                        txr->watchdog_time = ticks;

                        if (++first == adapter->num_tx_desc)
                                first = 0;
                        tx_buffer = &txr->tx_buffers[first];
                        tx_desc = &txr->tx_base[first];
                }

                ++ifp->if_opackets;

                last = tx_buffer->next_eop;
              if (last != −1) {
                        eop_desc = &txr->tx_base[last];
                        if (++last == adapter->num_tx_desc)
                                last = 0;
                        done = last;
                } else
                        break;
        }

        bus_dmamap_sync(txr->txdma.dma_tag, txr->txdma.dma_map,
            BUS_DMASYNC_PREWRITE);
...
```

这个 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) `while` 循环遍历 `first` 到 `last`，![](img/httpatomoreillycomsourcenostarchimages1137505.png) 释放它们的 mbufs 和 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 将它们的传输描述符置零。（`em(4)` 有一个有限的传输描述符数量。将描述符置零使其再次可用。）

这个 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `while` 循环 ![](img/httpatomoreillycomsourcenostarchimages1137507.png) 判断是否可以通过这个 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) `while` 循环释放另一个 mbuf 链。

这是 `em_txeof` 的最后一部分：

```
...
        txr->next_to_clean = first;

        if (!processed && ((ticks - txr->watchdog_time) > EM_WATCHDOG))
                txr->queue_status = EM_QUEUE_HUNG;

      if (txr->tx_avail > EM_MAX_SCATTER)
              ifp->if_drv_flags &= ˜IFF_DRV_OACTIVE;

        if (txr->tx_avail == adapter->num_tx_desc) {
                txr->queue_status = EM_QUEUE_IDLE;
              return (FALSE);
        }

      return (TRUE);
}
```

如果还有更多要回收的传输描述符，`em_txeof` 返回 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) `TRUE`；否则，它返回 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) `FALSE`。

如果可用传输描述符的数量 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 大于 `EM_MAX_SCATTER`，则可以传输数据包。

# 数据包传输后

每当接口发送一个数据包时，它会发送一个中断。自然地，这会导致其中断处理程序执行。以下是 `em(4)` 中执行的内容：

```
static void
em_msix_tx(void *arg)
{
        struct tx_ring *txr = arg;
        struct adapter *adapter = txr->adapter;
        bool more;

        ++txr->tx_irq;

        EM_TX_LOCK(txr);
        more = em_txeof(txr);
        EM_TX_UNLOCK(txr);
        if (more)
              taskqueue_enqueue(txr->tq, &txr->tx_task);
        else
                E1000_WRITE_REG(&adapter->hw, E1000_IMS, txr->ims);
}
```

### 注意

由于 MSI，`em(4)` 可以使用不同的中断处理程序来处理数据包传输和接收。

这个函数简单地 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 回收已使用的传输描述符。如果有更多描述符要回收，则将一个 `task` 结构排队。以下是该 `task` 结构的函数：

```
static void
em_handle_tx(void *context, int pending)
{
        struct tx_ring *txr = context;
        struct adapter *adapter = txr->adapter;
        struct ifnet *ifp = adapter->ifp;

        EM_TX_LOCK(txr);

      em_txeof(txr);
      em_start_locked(ifp, txr);
        E1000_WRITE_REG(&adapter->hw, E1000_IMS, txr->ims);

        EM_TX_UNLOCK(txr);
}
```

这个函数首先 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 回收任何已使用的传输描述符，之后，任何可能由于描述符不足而暂停的包将被 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) 传输。

# 结论

本章和第十六章介绍了网络设备和驱动程序的基础知识。如果你认真对待编写网络驱动程序，你应该全面复习`em(4)`。我建议从它的`device_attach`实现开始：`em_attach`。
