##############################################################
FISCO BCOS 质量管理
##############################################################

.. image:: _static/images/FISCO_BCOS_Logo.svg

FISCO BCOS 是一个稳定、高效、安全的区块链底层平台，经过多家机构、多个应用，长时间在生产环境运行的实际检验。

- `环境搭建 <https://github.com/FISCO-BCOS/FISCO-BCOS>`_   
- `连接 <http://mp.weixin.qq.com/mp/homepage?__biz=MzU5NTg0MjA4MA==&hid=9&sn=7edf9a62a2f45494671c91f0608db903&scene=18#wechat_redirect>`_  
- `兼容性 <https://mp.weixin.qq.com/s/hEn2rxqnqp0dF6OKH6Ua-A>`_ 
- `合约交易 <https://github.com/FISCO-BCOS/FISCO-BCOS/issues>`_ 



.. admonition:: 概览

   - 面向基于FISCO BCOS为底层平台的应用，完成质量管理中涉及到的测试框架内容，能保证业务线上的稳定运行。任何一个平台在发布使用前，都必须经过严格、周密的测试，尽可能做到在测试环境覆盖实际生产环境所有可能的场景，有条件的可以直接在生产环境上进行稳定性测试。
   - 确保质量目标与质量方针保持一致。质量方针为质量目标提供了制定和评审的框架，因此，质量目标应建立在质量方针的基础之上。我们可以采用从质量方针引出质量目标的方法，即在充分理解质量方针实质的基础上，将具体目标引出来，如：质量方针是：开拓创新，可以导出在一定时期内找出新产品作为目标并实现；质量方针是：顾客满意，可以导出顾客满意度、产品下一步目标，等等。

.. admonition:: 方针

    - 以稳定为前提、产品可用为基础、用户体验为向导，过程控制、持续改进、走向卓越。

.. admonition:: 目标

    - 杜绝一切区块链质量事故导致的业务中断，保障上层应用运行稳定、安全，持续提升用户使用满意度
   
.. admonition:: 实施

   质量实施以质量目标为依据，针对具体的项目展开操作，采用总分总的原则，从区块链全系统到具体某一模块及功能特性无差别饱和测试，再整合所有模块或者子系统为一个整体进行稳定性、性能等非功能性验证。

   - 从产品视角
   FISCO BCOS底层   
   
   WeBase平台     
   
   WeFamily应用    

   - 从操作层面
   功能性测试 
   
   自动化工厂 
   
   非功能性测试（DFX）  

.. toctree::
   :hidden:
   :maxdepth: 1
   
   docs/introduction.md
   docs/what_is_new.md
   docs/change_log/index.rst
   docs/installation.md
   docs/tutorial/index.rst
   docs/manual/index.rst
   docs/enterprise_tools/index.md
   docs/sdk/index.md
   docs/browser/browser.md
   docs/design/index.rst
   docs/api.md
   docs/faq.md
   docs/community.md
   docs/autotest.md
