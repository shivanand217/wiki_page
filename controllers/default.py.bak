# -*- coding: utf-8 -*-
# this file is released under public domain and you can use without limitations

# -------------------------------------------------------------------------
# This is a sample controller
# - index is the default action of any application
# - user is required for authentication and authorization
# - download is for downloading files uploaded in the db (does streaming)
# -------------------------------------------------------------------------

from gluon.contrib.markdown import WIKI as MARKDOWN

def index():
    """this controller returns a dictionary rendered by the view 
       it lists all wiki pages
    """
    
    pages = db().select(db.page.id,db.page.title,orderby=db.page.title)
    return dict(pages=pages)

@auth.requires_login()
def create():
    """creates a new empty wiki page"""
    form = SQLFORM(db.page).process(next=URL('index'))
    return dict(form=form)

def show():
    """shows a wiki page """
    this_page = db.page(request.args(0,cast=int)) or redirect(URL('index'))
    db.post.page_id.default = this_page.id
    form = SQLFORM(db.post).process() if auth.user else None
    pagecomments = db(db.post.page_id == this_page.id).select()
    return dict(page=this_page,comments=pagecomments,form=form)

@auth.requires_login()
def edit():
    """edit an existing wiki page """
    this_page = db.page(request.args(0,cast=int)) or redirect(URL('index'))
    form = SQLFORM(db.page,this_page).process(
    next = URL('show',args = request.args)
    )
    return dict(form=form)

@auth.requires_login()
def documents():
    page = db.page(request.args(0,cast=int)) or redirect(URL('index'))
    db.document.page_id.default = page.id
    db.document.page_id.writable=False
    grid = SQLFORM.grid(db.document.page_id == page.id,args=[page.id])
    return dict(page=page,grid=grid)

def user():
    """
    exposes:
    http://..../[app]/default/user/login
    http://..../[app]/default/user/logout
    http://..../[app]/default/user/register
    http://..../[app]/default/user/profile
    http://..../[app]/default/user/retrieve_password
    http://..../[app]/default/user/change_password
    http://..../[app]/default/user/bulk_register
    use @auth.requires_login()
        @auth.requires_membership('group name')
        @auth.requires_permission('read','table name',record_id)
    to decorate functions that need access control
    also notice there is http://..../[app]/appadmin/manage/auth to allow administrator to manage users
    """
    return dict(form=auth())


@cache.action()
def download():
    """
    allows downloading of uploaded files
    http://..../[app]/default/download/[filename]
    """
    return response.download(request, db)

def search():
    """an ajax wiki page"""
    return dict(
    form =FORM(INPUT(_id='keyword',
                     _name='keyword',
                     _onkeyup="ajax('callback',['keyword'],'target');")),
                     target_div = DIV(_id='target'))

def callback():
    """an ajax callback that returns a <ul> of links to wiki pages"""
    query = db.page.title.contains(request.vars.keyword)
    pages = db(query).select(orderby=db.page.title)
    links = [A(p.title, _href=URL('show',args=p.id)) for p in pages]
    return UL(*links)

def news():
    """ generating rss feed from the wiki pages """
    response.generic_patterns = ['.rss']
    
    pages=db().select(db.page.ALL,orderby = db.page.title)
    return dict(title='mywiki rss feed',
               link='http://127.0.0.1:8000/my_wiki/default/index',
               description='mywiki news',
               created_on=request.now,
               items=[dict(title=row.title,
                          link=URL('show',args=row.id, scheme=True, host=True, extension=False),
                          description=MARKMIN(row.body).xml(),
                          created_on = row.created_on
                          )for row in pages])

service = Service()

@service.xmlrpc
def find_by(keyword):
    """find the pages that contains keyword for XML-RPC"""
    return db(db.page.title.contains(keyword)).select().as_list()

def call():
    return service()

@auth.requires_login()
def manage_things():
    return SQLFORM.grid(db.thing)
