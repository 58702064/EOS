https://eosio.github.io/eos/group__contractdev.html
# How To Write Contracts
# ��α�д��Լ
Introduction to writing contracts for EOS.IO.  
�������Ϊ EOS.IO ��д��Լ��

## Detailed Description
## ��ϸ˵��  
### Background
### ����
EOS.IO contracts (aka applications) are deployed to a blockchain as pre-compiled Web Assembly (aka WASM). WASM is compiled from C/C++ using LLVM and clang, which means that you will require knowledge of C/C++ in order to develop your blockchain applications. While it is possible to develop in C, we strongly recommend that all developers use the EOS.IO C++ API which provides much stronger type safety and is generally easier to read.  
EOS.IO ��Լ��Ӧ�ó�����Ԥ����� Web Assembly��WASM������ʽ�������������ϡ�WASM ʹ�� C/C++ ��д��ͨ�� LLVM �� clang ���룬����ζ������Ҫ C/C++ ��֪ʶ���ܿ������������Ӧ�á���Ȼ������ C ��������������������ǿ���Ƽ����п�����ʹ�� EOS.IO �� C++ API������ǿ���Ͱ�ȫ�ģ�����ͨ���������Ķ���
### Application Structure
### Ӧ�ó���ṹ
EOS.IO applications are designed around event (aka message) handlers that respond to user actions. For example, a user might transfer tokens to another user. This event can be processed and potentially rejected by the sender, the receiver, and the currency application itself.  
EOS.IO Ӧ�ó�����Χ����Ӧ�û���Ϊ���¼�����Ϣ����������Ƶġ����磬һ���û�ת�� tokens ������һ���û�������¼����Ա����ͷ������շ����ߵ�ǰӦ�ó�������д�����߾ܾ���  
As an application developer you get to decide what actions users can take and which handlers may or must be called in response to those events.  
��ΪӦ�ó���Ŀ����ߣ�����Ҫ�����û����Դ���ʲô��Ϊ���Լ���Щ�¼�������Ӧ�û��߱��뱻��������Ӧ��Щ�¼���
### Entry Points
### ��ڵ�

EOS.IO applications have a apply which is like main in traditional applications:  
EOS.IO Ӧ�ó����ṩ���ƴ�ͳӦ�ó���� main ������ͬ���õ� apply ������Ϊ��ڵ㣺

```C
extern "C" {
   void init();
   void apply( uint64_t code, uint64_t action );
}
```

main is give the arguments code and action which uniquely identify every event in the system. For example, code could be a currency contract and action could be transfer. This event (code,action) may be passed to several contracts including the sender and receiver. It is up to your application to figure out what to do in response to such an event.  
main �������� code �� action ������������ϵͳ��ͨ��������������Ψһ��ʶÿ���¼������磬code ������һ���ֽ��Լ���� action ����ת�Ƶ���Ϊ������¼���code��action�����Ա����ݵ����������ߺͽ����ߵĶ����Լ�С����Ӧ�ó������Ҫ��ȷ�����Ӧ����¼���
init is another entry point that is called once immediately after loading the code. It is where you should perform one-time initialization of state.
init �Ǽ��ش����ᱻ�������ã���ֻ������һ�ε�����һ����ڵ㡣�����������ʵ��һ���Ե�״̬��ʼ����

### Example Apply Entry Handler
### apply ��ڴ�������ʾ��
Generally speaking, you should use your entry handler to dispatch events to functions that implement the majority of your logic and optionally reject events that your contract is unable or unwilling to accept.  
һ����˵����Ӧ��ʹ����ڴ�����ȥ�ַ��¼�����Ӧ�ķ�������Щ����ʵ����Ҫ���߼����Լ��ܾ���Լ��ʶ����߲�Ӧ�ý��ܵ��¼���

```C
extern "C" {
   void apply( uint64_t code, uint64_t action ) {
      if( code == N(currency) ) {
         if( action == N(transfer) )
            currency::apply_currency_transfer( currentMessage< currency::Transfer >() );
      } else {
         assert( false, "rejecting unexpected event" );
      }
   }
}
```
#### Note
#### ˵��
When defining your entry points it is required that they are placed in an extern "C" code block so that c++ name mangling does not get applied to the function.  
���㶨����ڵ�ʱ����Ҫ��������� extern "C" �������ʹ�� C++ ���ָı಻��Ӧ�õ��÷����ϡ�
