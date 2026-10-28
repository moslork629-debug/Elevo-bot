import discord
from discord import app_commands
from discord.ext import commands

intents = discord.Intents.default()
intents.message_content = True
intents.guilds = True
intents.members = True

bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_ready():
    try:
        synced = await bot.tree.sync()
        print(f"Logged in as {bot.user} and synced {len(synced)} commands.")
    except Exception as e:
        print(e)

# القائمة المنسدلة المطابقة لنظام Wicks في صورتك
class TicketSelect(discord.ui.Select):
    def __init__(self):
        options = [
            discord.SelectOption(label="الدعم الفني", description="للحصول على المساعدة والاستفسارات", emoji="🎫", value="support"),
            discord.SelectOption(label="شكوى", description="للتبليغ عن مشكلة أو عضو مخالف", emoji="⚠️", value="complaint"),
            discord.SelectOption(label="تقديم", description="للتقديم على الرتب أو الإدارة", emoji="📝", value="apply"),
            discord.SelectOption(label="طلب شراكة", description="لعقد شراكات مع السيرفرات الأخرى", emoji="🤝", value="partnership")
        ]
        super().__init__(placeholder="اختر خيار التذكرة...", min_values=1, max_values=1, options=options, custom_id="wick_ticket_select")

    async def callback(self, interaction: discord.Interaction):
        guild = interaction.guild
        ticket_type = self.values[0]
        
        # صلاحيات الروم (فقط العضو وصاحب البوت/الإدارة)
        overwrites = {
            guild.default_role: discord.PermissionOverwrite(view_channel=False),
            interaction.user: discord.PermissionOverwrite(view_channel=True, send_messages=True, read_messages=True),
            guild.me: discord.PermissionOverwrite(view_channel=True, send_messages=True, read_messages=True)
        }
        
        # إنشاء الروم باسم العضو
        ticket_channel = await guild.create_text_channel(
            name=f"ticket-{interaction.user.name}",
            overwrites=overwrites
        )
        
        embed_inside = discord.Embed(
            title="🔒 Secure Ticket System",
            description=f"أهلاً بك {interaction.user.mention}!\nنوع التذكرة المختارة: **{self.values[0]}**\nيرجى الانتظار، سيتم الرد عليك من قبل الإدارة قريباً.",
            color=discord.Color.dark_embed()
        )
        embed_inside.set_footer(text="Powered by Vetox System")
        
        close_view = WickCloseView()
        await ticket_channel.send(embed=embed_inside, view=close_view)
        await interaction.response.send_message(f"✅ تم إنشاء التذكرة بنجاح: {ticket_channel.mention}", ephemeral=True)

class TicketSelectView(discord.ui.View):
    def __init__(self):
        super().__init__(timeout=None)
        self.add_item(TicketSelect())

# زر إغلاق التذكرة
class WickCloseView(discord.ui.View):
    def __init__(self):
        super().__init__(timeout=None)

    @discord.ui.button(label="إغلاق التذكرة 🔒", style=discord.ButtonStyle.red, custom_id="wick_close_ticket")
    async def close_ticket(self, interaction: discord.Interaction, button: discord.ui.Button):
        await interaction.response.send_message("⚠️ جاري إغلاق التذكرة وحذف الروم...")
        await interaction.channel.delete()

# أمر إرسال اللوحة
@bot.tree.command(name="ticket-panel", description="إرسال لوحة التكت الاحترافية")
@app_commands.default_permissions(administrator=True)
async def ticket_panel(interaction: discord.Interaction):
    embed = discord.Embed(
        title="أهلاً بك 👋",
        description="كيف يمكننا مساعدتك؟ اختر من القائمة أدناه لفتح تذكرة جديدة.",
        color=discord.Color.from_rgb(43, 45, 49)
    )
    view = TicketSelectView()
    await interaction.channel.send(embed=embed, view=view)
    await interaction.response.send_message("تم إرسال اللوحة بنجاح!", ephemeral=True)

import os
bot.run(os.getenv("TOKEN"))
