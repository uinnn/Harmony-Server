# The official fork of Minecraft Server by Harmony

## Release
Atualmente o fork feito pelos desenvolvedores da Rede Harmony não está aberta ao público, entretanto, um dia possivelmente estará.

## O que há de novo?
Nesta seção, você encontrará uma lista abrangente (mas não completa) de todas as adições, remoções ou alterações feitas neste fork:

### 1) Remoção do sistema de iluminação
O sistema de iluminação do Minecraft foi removido, pois não é útil para nós, já que, na maioria dos casos, utilizamos ambientes com iluminação clara. No entanto, isso não significa que o jogo ficará completamente claro para o jogador. No lado do cliente, a funcionalidade original será mantida. A remoção do sistema de iluminação trouxe melhorias significativas em termos de desempenho e menor uso de memória RAM.

### 2) Bibliotecas embutidas
Bibliotecas fundamentais, como Kotlin e outras, estão embutidas no arquivo `.jar` final do servidor. Outras bibliotecas ainda serão baixadas e carregadas no diretório `libraries`. Isso deve, esperamos, reduzir os erros relacionados à duplicação de classes no `ClassLoader`.

### 3) Deobfuscação
Grande parte do código NMS foi deobfuscado para melhorar a legibilidade pelos desenvolvedores, o que facilita o uso e aumenta a produtividade, evitando a necessidade de pesquisar o que uma variável ou método faz. No entanto, é importante lembrar que a deobfuscação pode comprometer a compatibilidade com plugins de terceiros que utilizam NMS. Caso ocorram erros como `ClassNotFoundException`, `MethodNotFoundException` ou `FieldNotFoundException`, a deobfuscação deverá ser revisada para o caso específico.

### 4) Blocos customizados
A adição de blocos customizados é uma novidade para forks de servidores Minecraft. "Blocos customizados" refere-se ao comportamento do bloco no lado do servidor, com a possibilidade de alterar o modelo no lado do cliente. Por exemplo, um bloco pode ter o comportamento de uma folha, mas a aparência de um bloco de madeira. As possibilidades são vastas, com total suporte às funcionalidades de um bloco NMS, como: ticks, colisões, interações, entre outros. Reformulamos classes NMS relevantes para garantir a compatibilidade com esses blocos customizados. Algumas das classes alteradas incluem `Block` (para deobfuscação e registro) e `ChunkSection` (para armazenamento dos modelos dos blocos no lado do cliente).

### 5) Itens customizados
Assim como os blocos, agora é possível criar itens customizados. Itens customizados seguem a mesma lógica: comportamento no lado do servidor e um modelo modificado no lado do cliente. Eles oferecem uma gama de possibilidades com alto desempenho, se comparado ao método tradicional de criação de itens customizados. Exemplos incluem interações com entidades e blocos, além de ticks dentro de containers ou no inventário do jogador.

### 6) TileEntities customizados
TileEntities são blocos que armazenam dados específicos ou que exigem um processamento contínuo (tick), como baús, que armazenam itens, e Beacons, que aplicam efeitos periódicos em jogadores próximos. Facilitamos a criação de TileEntities customizados, oferecendo uma estrutura mais intuitiva.

### 7) Entidades customizadas
Embora o Minecraft já permita a criação de entidades NMS customizadas, a maior dificuldade sempre foi determinar qual entidade seria exibida no cliente. Com este fork, essa limitação foi removida. Agora, basta sobrescrever o método responsável pelo pacote de spawn da entidade para que o cliente veja a entidade correta. Isso possibilita, por exemplo, criar um `EnderCrystal` no lado do cliente com o comportamento de um `EnderDragon`.

### 8) Mais APIs nas classes do Bukkit
Adicionamos novos métodos às classes da API do Bukkit, facilitando o acesso a todas as funcionalidades oferecidas pelo Minecraft.

### 9) Otimizações
Implementamos diversas otimizações, tanto em nível macro quanto micro, no código. Uma lista detalhada dessas otimizações será divulgada em breve.

### 10) Melhorias no NBT do Minecraft
O sistema de NBT do Minecraft é notoriamente complicado, devido à deobfuscação e à falta de métodos claros para manipulação. Facilitamos o uso do NBT com novos métodos, otimizações e a substituição de coleções internas por implementações do FastUtil. Também adicionamos a capacidade de criar tipos personalizados de NBT. 

**Nota:** Tipos de NBT personalizados não podem ser utilizados em itens, pois o cliente não reconhece novas Tags personalizadas.

# Exemplos de código:
Uma leve amostra do código para a criação de blocos, 

## Blocos
```kt
// a object representing the custom block.
object ExampleBlock : Block(Material.STONE) { // this is NMS Material class, not Bukkit !
  init {
    setTickable(true)
  }  

  // allows the server to know if this block has a custom client model
  override fun hasCustomClientModel(): Boolean {
    return true
  }
  
  // returns the custom client model for this block
  // note that the return type is the id of the block state.
  // this is why we need to shift it by 4 (or multiply by 16)
  //
  // just a example for knowing it works
  //
  // stateId = (blockId shl 4) + (metadata and 15)
  //
  // so a block of "1:0" (stone), the stateId will be 16
  // a grass block, the stateId will be 32 (2:0)
  // a coarse dirt block, the stateId will be 49 (3:1)
  // 
  // etc... you must know it.
  //
  override fun getClientModel(original: Int): Int {
    return org.bukkit.Material.EMERALD_BLOCK.id shl 4
  }
  
  // example of functions...
  
  override fun onMove(world: World, pos: BlockPosition, data: IBlockData, entity: Entity) {
    println("entity moved above this custom block")
  }
  
  override fun updateTick(world: World, pos: BlockPosition, data: IBlockData, random: Random) {
    println("block have been random ticked")
  }
  
  // much more..
}
```

## Itens
```kt
// a custom item for placing our custom block
object CustomItem : ItemBlock(ExampleBlock) {
  
  // allows the server to know if this item has a custom client model
  override fun hasCustomClientModel(item: ItemStack): Boolean {
    return true
  }
  
  // gets the custom client model for this item
  override fun getClientItemModel(item: ItemStack): Int {
    return org.bukkit.Material.EMERALD.id
  }
  
  // gets the custom client metadata for this item
  override fun getClientMetadataModel(item: ItemStack): Int {
    return 0
  }
  
  // optionally you can override the amount of items showed in the client and their tag too..
  
  override fun getClientAmount(item: ItemStack): Int {
    return randomInt(1, 16) // will show a random amount between 1 and 16 on the client everytime item is updated
  }
  
  // this is just a example overriding the display name of the item.
  override fun getClientTag(item: ItemStack): NBTTagCompound? {
    val tag = item.tag.copy()
    val display = tag.getCompoundOrNull("display")
    display?.setString("Name", "§aI am a custom name from the server :)")
    return tag
  }
  
  override fun getMaxStackSize(): Int {
    return 8
  }
  
  override fun onTick(item: ItemStack, world: World, player: EntityPlayer, slot: Int, inHand: Boolean) {
    println("ticking from player inventory")
  }
  
  // and much more...
}
```
